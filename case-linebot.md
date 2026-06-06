# LINE Bot Backend System Portfolio

> **🔒 Disclaimer**: All code, API names, and database structures in this portfolio have been sanitized. This document is for technical capability demonstration only and does not contain any proprietary company information.

## Project Overview

A complete LINE Bot backend system developed independently, serving **24,000+ users** with member management and product stamp collection features.

> **Context for international readers**: [LINE](https://line.me/) is the dominant messaging platform in Taiwan (used by 90%+ of the population), similar to WhatsApp in Europe or WeChat in China. LINE Official Accounts are widely used by businesses for customer engagement, marketing, and service delivery.

| Item | Details |
| --- | --- |
| **Role** | Backend Developer (sole developer for all LINE API integration and backend logic) |
| **Tech Stack** | .NET 8 Web API, Entity Framework Core, SQL Server, LINE Messaging API, LIFF, Serilog, GitHub Actions CI/CD |
| **User Scale** | 24,000+ LINE friends |
| **System Scale** | 35 API endpoints, 10 controllers |
| **Database** | 3 independent databases integrated |
| **Architecture** | Layered architecture + Dependency Injection + Multi-database transactions |

---

## Business Problem Solved

Built a one-stop member engagement and marketing platform through LINE Official Account, establishing a complete member ecosystem to support product promotion strategies, improving member engagement and operational efficiency.

---

## 1. Project Background & Objectives

I was responsible for the entire LINE API integration and all backend logic development from scratch, covering Webhook verification, member binding, OTP verification, Flex Message delivery, stamp collection and coupon redemption. I worked closely with the design and planning teams to ensure the system met design specifications and business requirements.

**Core Features:**

1. **Webhook Event Processing:** Securely receive and verify various LINE events, including text messages, Rich Menu button clicks, etc.
2. **Member Binding & OTP Verification:** Verify identity through SMS OTP, bind LINE users to the existing member system, ensuring data accuracy and security.
3. **Product Stamp Collection:** Scan QR Codes to accumulate stamp points; when reaching the 12-point threshold, automatically redeem coupons with real-time Flex Message notifications. Promotes product sales through QR Code scanning.
4. **Dynamic Rich Menu Switching:** Dynamically switch the LINE Official Account's Rich Menu based on user status or business needs, providing a personalized experience.
5. **Coupon Management:** Display and use redeemed coupons, managing the complete lifecycle from redemption to usage.
6. **Real-time Flex Message Notifications:** Multiple touchpoints throughout the system use rich Flex Messages to deliver information clearly and enhance user experience.

---

## 2. Technical Architecture & Tools

| Category | Technology / Tool |
| --- | --- |
| Language | C# |
| **Framework** | ASP.NET Core 8 WebAPI |
| **Database** | SQL Server (EF Core Scaffold + Stored Procedures) |
| **API Security** | LINE signature verification, Swagger IP filtering, duplicate request prevention |
| **Integrations** | LINE Messaging API, LIFF, SMS provider API, LINE Notify (later replaced with Discord) |
| **Messaging** | Flex Message (JSON templates + `PushFlexMessageAsync`) |
| **DI** | ASP.NET Core DI for all services |
| **CI/CD** | GitHub Actions: Build + Unit Tests, Dev/Test deploy, Release deploy, Rollback workflow |

---

## 3. Database Design & ER Architecture

- **Led the design** of multiple business tables in SSMS (SQL Server Management Studio), including User_Profile, Collection_Record, Collection_Redeem, QrCode_Record, Event_Record, etc. Defined primary keys, indexes, foreign keys, and default values while coordinating naming conventions and performance requirements.
- Used one-time reverse engineering (Scaffold-DbContext) to quickly map database structures to EF Core entity models, significantly improving development efficiency.

**Generalized Table Structure (sanitized)**

| Table Name | Description |
| --- | --- |
| **User_Profile** | Stores user personal data, binding status, member ID |
| **Collection_Record** | Records each QR Code scan with stamp points and redemption status |
| **QrCode_Record** | Manages randomly generated QR Codes |
| **Coupon_Redeem** | Stamp collection coupon redemption records |
| **Event_Record** | Logs all raw Webhook event payloads |
| **Link_History** | Records each user binding history |
| **Coupon** | Redeemed coupon data including code, expiry date, usage status |
| **SmsLog** | Records OTP SMS delivery results |

![figure-01-er-model](https://hackmd.io/_uploads/HkxVf8LuDeg.jpg)
*Figure 1: Database Entity Relationship Diagram — Complete data structure of the member system, stamp collection system, and coupon system*

---

## 4. Core Feature Implementations

### 4.1 Webhook Event Processing

![figure-02-webhook-flow](https://hackmd.io/_uploads/SJA5ILODge.png)
*Figure 2: Webhook Event Processing Flow — From receiving LINE platform events to completion, including security verification, event routing, async processing, and error handling*

> This module receives and processes Webhook requests from the LINE platform, featuring security verification, event routing, data storage, and external system sync. All processing is asynchronous with error protection and logging to ensure stable, uninterrupted operation.

**Implementation Highlights:**

- **Signature Verification & High Availability Response**
    - Each incoming request is verified via HMAC-SHA256 against `X-Line-Signature` to confirm it originates from LINE's official servers.
    - Returns HTTP 200 immediately upon verification to prevent LINE from resending events due to processing delays.
- **Modular Event Routing & Async Execution:**
    - Routes events by type (Message, Postback, Follow, Unfollow) to corresponding handlers.
    - All business logic runs asynchronously to avoid blocking the main thread, ensuring high concurrency.

#### Implementation Details

##### Security Verification

```csharp
[HttpPost("linebot-webhook")]
public async Task<IActionResult> ProcessWebhookEvent()
{
    try
    {
        var body = await ReadRequestBody();
        var signature = Request.Headers["X-Line-Signature"].FirstOrDefault();
        
        // HMAC-SHA256 signature verification
        if (!VerifySignature(body, signature))
        {
            _logger.LogWarning("Invalid LINE signature");
            return Unauthorized();
        }
        
        // Respond with 200 immediately to prevent LINE from resending
        var response = Ok();
        
        // Process events asynchronously without blocking the response
        _ = Task.Run(async () => await ProcessIncomingEvents(body));
        
        return response;
    }
    catch (Exception ex)
    {
        _logger.LogError(ex, "Webhook processing exception");
        return StatusCode(500);
    }
}

private bool VerifySignature(string body, string signature)
{
    var channelSecret = _configuration["LineBot:ChannelSecret"];
    var hash = Convert.ToBase64String(
        new HMACSHA256(Encoding.UTF8.GetBytes(channelSecret))
        .ComputeHash(Encoding.UTF8.GetBytes(body))
    );
    return signature == hash;
}
```

##### Event Routing

```csharp
private async Task ProcessIncomingEvents(string requestBody)
{
    var webhookEvents = JsonSerializer.Deserialize<WebhookEventContainer>(requestBody);
    
    foreach (var eventObj in webhookEvents.Events)
    {
        // Log raw event
        await LogIncomingEvent(eventObj);
        
        // Route by event type
        switch (eventObj.Type)
        {
            case "message":
                await HandleMessageEvent(eventObj);
                break;
            case "postback":
                await HandlePostbackEvent(eventObj);
                break;
            case "follow":
                await HandleFollowEvent(eventObj);
                break;
            case "unfollow":
                await HandleUnfollowEvent(eventObj);
                break;
        }
    }
}
```

---

### 4.2 Member Binding & OTP Verification

![figure-03-otp-binding-flow](https://hackmd.io/_uploads/SJq0ULODxe.png)
*Figure 3: OTP Verification & Member Binding Flow — From phone verification to account selection*

> Technical Background: Implements secure binding between LINE users and the existing member system, using SMS OTP verification to ensure identity authenticity. Supports both new member registration and existing member binding flows.

#### Implementation Highlights

##### Multi-layer Identity Verification & Anti-abuse Protection

- **Cryptographic random number generation**: Uses `RandomNumberGenerator.GetInt32()` instead of basic `Random` class, meeting high security standards for OTP unpredictability.
- **Rate limiting (5 attempts per 24 hours)**: Prevents brute-force attacks through timestamp comparison and counter mechanism, protecting SMS resources and system security.
- **OTP state management**: Immediately clears verification code and resets counter upon successful verification to prevent reuse.
- **Duplicate binding prevention**: Checks whether a member ID is already bound to another LINE account, ensuring one-to-one relationships.

##### Third-party Service Integration & Error Handling

- **SMS service integration**: Complete handling of SMS API response parsing, UTF-8 encoding, status code validation, and delivery history logging.
- **LINE Messaging API integration**: Automatically sends Flex Message notifications for binding results, enhancing user experience.
- **Standardized error response format**: Unified API response structure for consistent frontend error handling.

##### Complete Business Flow Support

- **Existing member binding flow**: OTP verification → Multi-account selection → Binding confirmation → Success notification
- **New member registration flow**: Member ID generation → Cross-system registration → Auto-binding → Setup complete
- **Store referral system**: Supports store code parameters to track referral sources and establish binding history
- **Binding history tracking**: Complete audit trail of the binding process for analytics and customer support

##### Cross-database Transaction Processing & Data Consistency

- **Three-system data synchronization**: Member system (basic data) + E-commerce system (identity) + LINE system (binding status)
- **Staged transaction processing**: Sequential updates across databases with login status management to ensure data consistency
- **Auto member ID generation**: Smart ID generator with format validation and sequence incrementing

##### Multi-account Business Logic

- **One-to-many relationship handling**: Supports the business scenario of one phone number mapping to multiple member accounts
- **Account selection interface**: Returns all bindable accounts for user selection after OTP verification
- **Secure switching mechanism**: Only allows switching between accounts with the same phone number, preventing cross-account malicious binding

#### Code Examples

##### 1. Secure OTP Generation with Rate Limiting

```csharp
[HttpPost("SendVerificationCode")]
public async Task<IActionResult> SendVerificationCode([FromBody] VerificationCodeRequestDto request)
{
    var response = new ApiResponse<VerificationCodeResponseDto>
    {
        IsSuccess = false,
        MessageCode = 9000,
        Message = "An unexpected error occurred",
        RequestData = request
    };

    try
    {
        var userProfile = await _context.UserProfiles
            .FirstOrDefaultAsync(p => p.UserId == request.UserId);

        // Rate limiting: Check if 5-attempt limit within 24 hours is reached
        if (userProfile.CodeSentTime.HasValue &&
            userProfile.CodeSentTime.Value.AddHours(24) > DateTime.Now &&
            userProfile.CodeAttemptCount >= 5)
        {
            response.MessageCode = 2002;
            response.Message = "Exceeded maximum send attempts within 24 hours";
            return BadRequest(response);
        }

        // Cryptographically secure random number generator for 4-digit OTP
        var otpCode = RandomNumberGenerator.GetInt32(1000, 9999).ToString();

        // Prepare SMS content and call SMS service
        var smsRequest = new SmsModel
        {
            PhoneNumber = request.PhoneNumber,
            Message = $"[Company] Your verification code is {otpCode}. Do not share this code.",
            Status = 0,
            CreatedAt = DateTime.Now
        };

        // Send SMS and update delivery record
        bool isSent = await _smsService.SendSmsAsync(smsRequest);
        if (isSent)
        {
            userProfile.CurrentVerificationCode = otpCode;
            userProfile.CodeSentTime = DateTime.Now;
            userProfile.CodeAttemptCount = (userProfile.CodeAttemptCount ?? 0) + 1;
            await _context.SaveChangesAsync();

            response.IsSuccess = true;
            response.MessageCode = 1000;
            response.Message = "SMS sent successfully";
            response.ResponseData = new VerificationCodeResponseDto
            {
                OtpCount = userProfile.CodeAttemptCount
            };

            return Ok(response);
        }

        response.MessageCode = 5000;
        response.Message = "SMS delivery failed";
        return BadRequest(response);
    }
    catch (Exception ex)
    {
        _logger.LogError(ex, "Error sending OTP: UserId={UserId}", request.UserId);
        return BadRequest(response);
    }
}
```

##### 2. Cross-database Transaction Processing

```csharp
[HttpPost("LinkMemberAccount")]
public async Task<IActionResult> LinkMemberAccount([FromBody] LinkAccountRequestDto request)
{
    // Begin database transaction to ensure cross-database operation consistency
    using var transaction = await _context.Database.BeginTransactionAsync();
    try
    {
        // Step 1: Query user's basic data from the member system
        var member = await _context.Members
            .FirstOrDefaultAsync(m => m.MemberId == profile.InternalMemberId);

        if (member != null)
        {
            // Step 2: Query corresponding member data from e-commerce system (linked by phone number)
            var ecommerceMember = await _context.Members
                .FirstOrDefaultAsync(m => m.Account == member.Phone);

            if (ecommerceMember != null)
            {
                // Step 3: Update LINE binding status in e-commerce system
                ecommerceMember.IsLineBound = true;
                ecommerceMember.LineBindTime = DateTime.Now;
                ecommerceMember.LineUserId = request.LineUserId;
                _context.Members.Update(ecommerceMember);

                // Step 4: Update login status (DeviceType=3 means LINE login)
                var loginStatus = new LoginStatus
                {
                    MemberId = ecommerceMember.MemberId,
                    DeviceType = 3,
                    Status = 1, // 1=logged in
                    LoginTime = DateTime.Now
                };
                _context.LoginStatuses.Add(loginStatus);

                // Step 5: Update user data in LINE system
                var lineProfile = await _context.UserProfiles
                    .FirstOrDefaultAsync(p => p.UserId == request.LineUserId);
                
                if (lineProfile != null)
                {
                    lineProfile.IsBound = true;
                    lineProfile.AccountCode = ecommerceMember.AccountCode;
                    lineProfile.BindTime = DateTime.Now;
                    _context.UserProfiles.Update(lineProfile);
                }

                // Commit all changes
                await _context.SaveChangesAsync();
                await _context.SaveChangesAsync();
                await _context.SaveChangesAsync();
                await transaction.CommitAsync();

                // Send binding success notification
                await _lineMessageService.SendLinkSuccessMessage(request.LineUserId, member.Name);

                return Ok(new { Success = true, Message = "Member binding successful" });
            }
        }

        await transaction.RollbackAsync();
        return BadRequest(new { Success = false, Message = "Binding failed, please check member data" });
    }
    catch (Exception ex)
    {
        await transaction.RollbackAsync();
        _logger.LogError(ex, "Error during member binding");
        return StatusCode(500, new { Success = false, Message = "System error" });
    }
}
```

---

### 4.3 Member Data Query & Account Switching

#### Member Account Management System

![figure-04-member-account-view](https://hackmd.io/_uploads/S1oevI_wxg.png)
*Figure 4: Member Account Data View — Displaying member profile and e-commerce account management integration*

![figure-05-member-points](https://hackmd.io/_uploads/B1lfDLODge.png)
*Figure 5: Member Points System — Points query, discount application, and cross-platform integration*

![figure-06-account-switching](https://hackmd.io/_uploads/ByKQD8dvgg.png)
*Figure 6: Account Switching Interface — Multi-account selection, vision data display (left/right eye prescription), and secure switching*

#### Feature Overview

- **Multi-account support**: Query and preview multiple member accounts under the same phone number
- **Integrated account data**: Consolidates member profile, points, coupons, and order information
- **Secure switching mechanism**: Phone number verification ensures safe account switching
- **Real-time data sync**: Immediately updates all related business data after account switch

#### Core Technical Implementation

##### 1. Multi-account Query & Display

```csharp
[HttpGet("GetAccountsByPhone")]
public async Task<IActionResult> GetAccountsByPhone([FromQuery] string phoneNumber)
{
    try
    {
        // Query all accounts for this phone number from the member system
        var members = await _context.Members
            .Where(m => m.Phone == phoneNumber && m.IsActive)
            .Select(m => new AccountSummaryDto
            {
                AccountCode = m.AccountCode,
                Name = m.Name,
                Phone = m.Phone,
                RegisterDate = m.CreateTime,
                // Query point balance for this member
                PointBalance = _context.PointRecords
                    .Where(p => p.AccountCode == m.AccountCode)
                    .Sum(p => p.Points),
                // Query unused coupon count
                CouponCount = _context.Coupons
                    .Count(c => c.AccountCode == m.AccountCode && !c.IsUsed)
            })
            .ToListAsync();

        return Ok(new ApiResponse<List<AccountSummaryDto>>
        {
            IsSuccess = true,
            ResponseData = members,
            Message = $"Found {members.Count} member accounts"
        });
    }
    catch (Exception ex)
    {
        _logger.LogError(ex, "Failed to query member accounts: Phone={Phone}", phoneNumber);
        return StatusCode(500, new { Message = "Query failed" });
    }
}
```

##### 2. Secure Account Switching Mechanism

```csharp
[HttpPost("ChangeActiveAccount")]
public async Task<IActionResult> ChangeActiveAccount([FromBody] ChangeAccountRequestDto request)
{
    using var transaction = await _context.Database.BeginTransactionAsync();
    try
    {
        // Verify target member account belongs to the same phone number
        var currentProfile = await _context.UserProfiles
            .FirstOrDefaultAsync(p => p.UserId == request.LineUserId);
            
        var targetMember = await _context.Members
            .FirstOrDefaultAsync(m => m.AccountCode == request.TargetAccountCode);

        if (currentProfile.Phone != targetMember.Phone)
        {
            return BadRequest(new { Message = "Cannot switch to an account with a different phone number" });
        }

        // Update the member ID bound to the LINE user
        currentProfile.AccountCode = request.TargetAccountCode;
        currentProfile.LastSwitchTime = DateTime.Now;
        
        // Update login status in e-commerce system
        var loginStatus = await _context.LoginStatuses
            .FirstOrDefaultAsync(l => l.LineUserId == request.LineUserId);
            
        if (loginStatus != null)
        {
            loginStatus.MemberId = targetMember.MemberId;
            loginStatus.LoginTime = DateTime.Now;
        }

        // Record account switch history
        var switchHistory = new AccountSwitchHistory
        {
            LineUserId = request.LineUserId,
            FromAccountCode = currentProfile.AccountCode,
            ToAccountCode = request.TargetAccountCode,
            SwitchTime = DateTime.Now,
            IpAddress = GetClientIpAddress()
        };
        
        await _context.AccountSwitchHistories.AddAsync(switchHistory);
        await _context.SaveChangesAsync();
        await _context.SaveChangesAsync();
        await transaction.CommitAsync();

        // Send switch success notification
        await _lineMessageService.SendSwitchNotification(
            request.LineUserId, targetMember.Name);

        return Ok(new { Success = true, Message = "Account switch successful" });
    }
    catch (Exception ex)
    {
        await transaction.RollbackAsync();
        _logger.LogError(ex, "Account switch failed");
        return StatusCode(500, new { Message = "Switch failed" });
    }
}
```

##### 3. Integrated Data Query with Parallel Execution

```csharp
[HttpGet("GetAccountDetails")]
public async Task<IActionResult> GetAccountDetails([FromQuery] string memberCode)
{
    try
    {
        // Parallel queries across systems to improve response time
        var memberTask = _context.Members
            .FirstOrDefaultAsync(m => m.AccountCode == memberCode);
            
        var pointsTask = _context.PointRecords
            .Where(p => p.AccountCode == memberCode)
            .SumAsync(p => p.Points);
            
        var couponsTask = _context.Coupons
            .Where(c => c.AccountCode == memberCode && !c.IsUsed)
            .CountAsync();
            
        var ordersTask = _context.Orders
            .Where(o => o.AccountCode == memberCode)
            .OrderByDescending(o => o.CreateTime)
            .Take(5)
            .ToListAsync();

        // Wait for all queries to complete
        await Task.WhenAll(memberTask, pointsTask, couponsTask, ordersTask);

        var memberInfo = new AccountDetailDto
        {
            AccountCode = memberTask.Result.AccountCode,
            Name = memberTask.Result.Name,
            Phone = memberTask.Result.Phone,
            Email = memberTask.Result.Email,
            PointBalance = pointsTask.Result,
            CouponCount = couponsTask.Result,
            RecentOrders = ordersTask.Result
        };

        return Ok(new ApiResponse<AccountDetailDto>
        {
            IsSuccess = true,
            ResponseData = memberInfo
        });
    }
    catch (Exception ex)
    {
        _logger.LogError(ex, "Failed to query account details: AccountCode={AccountCode}", memberCode);
        return StatusCode(500, new { Message = "Query failed" });
    }
}
```

---

### 4.4 Product Stamp Collection System

![figure-07-stamp-collection-flow](https://hackmd.io/_uploads/HyIUwI_wxe.png)
*Figure 7: Stamp Collection Flow — 4-step process from Rich Menu trigger to QR Code scan completion*

![figure-08-stamp-completion-flow](https://hackmd.io/_uploads/BkUww8dPle.png)
*Figure 8: Stamp Redemption Flow — Coupon redemption process after collecting 12 points*

#### Feature Overview

- **QR Code stamp scanning**: Supports product packaging barcode scanning for promotional stamp collection campaigns
- **Duplicate scan prevention**: Each QR Code can only be used once to prevent duplicate stamp collection
- **Progress notifications**: Alerts users when reaching the 12-point threshold for redemption
- **Real-time progress tracking**: Provides current stamp progress and remaining points query
- **Campaign period control**: Supports start and end time management for stamp collection campaigns

#### Core Technical Implementation

##### 1. QR Code Scan & Validation

```csharp
[HttpPost("ProcessStampScan")]
public async Task<IActionResult> ProcessStampScan([FromBody] StampScanRequestDto request)
{
    using var transaction = await _context.Database.BeginTransactionAsync();
    try
    {
        // Validate QR Code is valid and unused
        var qrCode = await _context.QrCodeRecords
            .FirstOrDefaultAsync(q => q.Code == request.QrCode && q.IsActive);

        if (qrCode == null)
        {
            return BadRequest(new ApiResponse 
            { 
                IsSuccess = false, 
                Message = "Invalid QR Code" 
            });
        }

        if (qrCode.IsUsed)
        {
            return BadRequest(new ApiResponse 
            { 
                IsSuccess = false, 
                Message = "This QR Code has already been used" 
            });
        }

        // Check campaign period
        if (DateTime.Now < qrCode.StartDate || DateTime.Now > qrCode.EndDate)
        {
            return BadRequest(new ApiResponse 
            { 
                IsSuccess = false, 
                Message = "Campaign period has ended" 
            });
        }

        // Mark QR Code as used
        qrCode.IsUsed = true;
        qrCode.UsedTime = DateTime.Now;
        qrCode.UserId = request.LineUserId;

        // Create stamp collection record
        var stampRecord = new CollectionRecord
        {
            UserId = request.LineUserId,
            QrCodeId = qrCode.Id,
            ScanTime = DateTime.Now,
            Points = qrCode.Points, // Typically 1 point
            ActivityId = qrCode.ActivityId
        };

        await _context.CollectionRecords.AddAsync(stampRecord);

        // Calculate user's current total points
        var currentPoints = await _context.CollectionRecords
            .Where(s => s.UserId == request.LineUserId && 
                       s.ActivityId == qrCode.ActivityId)
            .SumAsync(s => s.Points);

        var response = new StampScanResponseDto
        {
            Success = true,
            CurrentPoints = currentPoints,
            RequiredPoints = 12,
            Message = $"Stamp collected! Currently at {currentPoints} points"
        };

        // Check if redemption threshold is reached
        if (currentPoints >= 12)
        {
            response.Message = "Congratulations! You've reached 12 points. Redeem your coupon in the member area";
            response.CanRedeem = true;
        }

        await _context.SaveChangesAsync();
        await transaction.CommitAsync();

        // Send stamp collection success notification
        await _lineMessageService.SendCollectionNotification(
            request.LineUserId, currentPoints, 12);

        return Ok(new ApiResponse<StampScanResponseDto>
        {
            IsSuccess = true,
            ResponseData = response
        });
    }
    catch (Exception ex)
    {
        await transaction.RollbackAsync();
        _logger.LogError(ex, "QR Code scan failed: UserId={UserId}, QrCode={QrCode}", 
            request.LineUserId, request.QrCode);
        return StatusCode(500, new { Message = "Scan failed" });
    }
}
```

##### 2. Stamp Redemption Processing

```csharp
private async Task<bool> ProcessRedemption(string userId, int activityId)
{
    try
    {
        // Check if already redeemed
        var existingRedeem = await _context.CollectionRedeems
            .FirstOrDefaultAsync(r => r.UserId == userId && 
                                    r.ActivityId == activityId);

        if (existingRedeem != null)
        {
            return false; // Already redeemed
        }

        // Get user's member ID
        var userProfile = await _context.UserProfiles
            .FirstOrDefaultAsync(p => p.UserId == userId);

        // Create coupon
        var coupon = new Coupon
        {
            CouponCode = GenerateCouponCode(),
            AccountCode = userProfile.AccountCode,
            CouponType = "STAMP_REDEEM",
            DiscountAmount = 100, // 100 TWD discount
            ExpiryDate = DateTime.Now.AddMonths(3),
            IsUsed = false,
            CreateTime = DateTime.Now,
            ActivityId = activityId
        };

        await _context.Coupons.AddAsync(coupon);

        // Record redemption history
        var redeemRecord = new CollectionRedeem
        {
            UserId = userId,
            ActivityId = activityId,
            CouponId = coupon.Id,
            RedeemTime = DateTime.Now,
            RequiredPoints = 12,
            RedeemType = "MANUAL"
        };

        await _context.CollectionRedeems.AddAsync(redeemRecord);

        // Send redemption success Flex Message
        await _lineMessageService.SendRedeemSuccessMessage(
            userId, coupon.CouponCode, coupon.DiscountAmount, coupon.ExpiryDate);

        return true;
    }
    catch (Exception ex)
    {
        _logger.LogError(ex, "Stamp redemption processing failed: UserId={UserId}", userId);
        return false;
    }
}

private string GenerateCouponCode()
{
    // Generate unique coupon code: prefix + timestamp + random number
    var timestamp = DateTime.Now.ToString("yyyyMMddHHmm");
    var randomPart = RandomNumberGenerator.GetInt32(1000, 9999);
    return $"STAMP{timestamp}{randomPart}";
}
```

##### 3. Stamp Progress Query

```csharp
[HttpGet("GetStampProgress")]
public async Task<IActionResult> GetStampProgress([FromQuery] string userId, [FromQuery] int activityId)
{
    try
    {
        // Query all stamp records for this user in this campaign
        var stampRecords = await _context.CollectionRecords
            .Where(s => s.UserId == userId && s.ActivityId == activityId)
            .OrderByDescending(s => s.ScanTime)
            .Select(s => new CollectionRecordDto
            {
                ScanTime = s.ScanTime,
                Points = s.Points,
                QrCodeInfo = s.QrCode.ProductName
            })
            .ToListAsync();

        var totalPoints = stampRecords.Sum(s => s.Points);
        var requiredPoints = 12;
        var isCompleted = totalPoints >= requiredPoints;

        // Check if already redeemed
        var hasRedeemed = await _context.CollectionRedeems
            .AnyAsync(r => r.UserId == userId && r.ActivityId == activityId);

        var progress = new CollectionProgressDto
        {
            UserId = userId,
            ActivityId = activityId,
            CurrentPoints = totalPoints,
            RequiredPoints = requiredPoints,
            IsCompleted = isCompleted,
            HasRedeemed = hasRedeemed,
            CollectionRecords = stampRecords,
            ProgressPercentage = Math.Min(100, (totalPoints * 100) / requiredPoints)
        };

        return Ok(new ApiResponse<CollectionProgressDto>
        {
            IsSuccess = true,
            ResponseData = progress
        });
    }
    catch (Exception ex)
    {
        _logger.LogError(ex, "Failed to query stamp progress: UserId={UserId}", userId);
        return StatusCode(500, new { Message = "Query failed" });
    }
}
```

---

### 4.5 Coupon Management System

![figure-09-activity-coupons](https://hackmd.io/_uploads/Hk0dDUOwxx.png)
*Figure 9: Campaign Coupon Management Interface — Coupon status management and usage features*

#### E-commerce System Integration

![figure-10-member-orders](https://hackmd.io/_uploads/rJDjDLuvex.png)
*Figure 10: Member Order Query Integration — Cross-platform order data integration and query*

#### Feature Overview

- **Multi-type coupon support**: Manages discount coupons, redemption coupons, campaign coupons, and other types
- **Coupon status tracking**: Complete lifecycle tracking from issuance to usage and expiry
- **Usage restriction controls**: Supports usage count limits, expiry dates, and applicable conditions
- **Real-time inventory management**: Dynamic coupon inventory management to prevent over-issuance and duplicate usage
- **Complex data processing**: DataTable handling for dynamic structure stored procedure results

#### Core Technical Implementation

##### 1. Transaction-protected Redemption Mechanism

```csharp
[HttpPost("RedeemStampsCoupon")]
public async Task<IActionResult> ProcessCouponRedemption([FromBody] CouponRedemptionRequestDto request)
{
    using var transaction = await _context.Database.BeginTransactionAsync();
    try
    {
        // Verify stamp points are sufficient
        var userStamps = await _context.CollectionRecords
            .Where(s => s.UserId == request.UserId && !s.IsUsed)
            .SumAsync(s => s.Points);

        if (userStamps < request.RequiredPoints)
        {
            return BadRequest(new ApiResponse
            {
                IsSuccess = false,
                MessageCode = 4001,
                Message = $"Insufficient points. Currently have {userStamps}, need {request.RequiredPoints}"
            });
        }

        // Check if same coupon type already redeemed
        var existingCoupon = await _context.CouponRedeems
            .FirstOrDefaultAsync(cr => cr.UserId == request.UserId && 
                                      cr.CouponType == request.CouponType &&
                                      cr.IsActive);

        if (existingCoupon != null)
        {
            return BadRequest(new ApiResponse
            {
                IsSuccess = false,
                MessageCode = 4002,
                Message = "You have already redeemed this type of coupon"
            });
        }

        // Call stored procedure for complex redemption logic
        var parameters = new[]
        {
            new SqlParameter("@UserId", request.UserId),
            new SqlParameter("@CouponType", request.CouponType),
            new SqlParameter("@RequiredPoints", request.RequiredPoints),
            new SqlParameter("@Result", SqlDbType.Int) { Direction = ParameterDirection.Output }
        };

        await _context.Database.ExecuteSqlRawAsync(
            "EXEC sp_ProcessRedemption @UserId, @CouponType, @RequiredPoints, @Result OUTPUT", 
            parameters);

        var result = (int)parameters[3].Value;

        if (result == 1) // Redemption successful
        {
            // Mark stamp records as redeemed
            var stampsToRedeem = await _context.CollectionRecords
                .Where(s => s.UserId == request.UserId && !s.IsUsed)
                .Take(request.RequiredPoints)
                .ToListAsync();

            stampsToRedeem.ForEach(s => s.IsUsed = true);

            await _context.SaveChangesAsync();
            await transaction.CommitAsync();

            // Send redemption success notification
            await _lineMessageService.SendCouponSuccessMessage(request.UserId);

            return Ok(new ApiResponse
            {
                IsSuccess = true,
                MessageCode = 1000,
                Message = "Redemption successful!"
            });
        }
        else
        {
            await transaction.RollbackAsync();
            return BadRequest(new ApiResponse
            {
                IsSuccess = false,
                MessageCode = 4003,
                Message = "Redemption failed, please try again later"
            });
        }
    }
    catch (Exception ex)
    {
        await transaction.RollbackAsync();
        _logger.LogError(ex, "Coupon redemption failed: UserId={UserId}", request.UserId);
        return StatusCode(500, new ApiResponse
        {
            IsSuccess = false,
            MessageCode = 9000,
            Message = "System error"
        });
    }
}
```

##### 2. Stored Procedure Integration & DataTable Processing

```csharp
[HttpGet("GetMyCoupons")]
public async Task<IActionResult> GetMyCoupons([FromQuery] CouponListRequestDto request)
{
    try
    {
        var parameters = new[]
        {
            new SqlParameter("@UserId", request.UserId),
            new SqlParameter("@CouponStatus", request.Status ?? (object)DBNull.Value),
            new SqlParameter("@PageIndex", request.PageIndex),
            new SqlParameter("@PageSize", request.PageSize)
        };

        // Use DataTable to receive dynamic stored procedure results
        using var command = _context.Database.GetDbConnection().CreateCommand();
        command.CommandText = "sp_GetUserCouponList";
        command.CommandType = CommandType.StoredProcedure;
        command.Parameters.AddRange(parameters);

        await _context.Database.OpenConnectionAsync();
        using var reader = await command.ExecuteReaderAsync();
        
        var dataTable = new DataTable();
        dataTable.Load(reader);

        // Convert DataTable to strongly-typed objects
        var coupons = new List<CouponInfoDto>();
        foreach (DataRow row in dataTable.Rows)
        {
            coupons.Add(new CouponInfoDto
            {
                CouponId = row["CouponId"].ToString(),
                CouponCode = row["CouponCode"].ToString(),
                CouponName = row["CouponName"].ToString(),
                DiscountAmount = Convert.ToDecimal(row["DiscountAmount"]),
                ExpiryDate = Convert.ToDateTime(row["ExpiryDate"]),
                IsUsed = Convert.ToBoolean(row["IsUsed"]),
                UsageConditions = row["UsageConditions"].ToString(),
                CouponStatus = row["Status"].ToString()
            });
        }

        var response = new ApiResponse<List<CouponInfoDto>>
        {
            IsSuccess = true,
            ResponseData = coupons,
            Message = $"Found {coupons.Count} coupons"
        };

        return Ok(response);
    }
    catch (Exception ex)
    {
        _logger.LogError(ex, "Failed to query user coupons: UserId={UserId}", request.UserId);
        return StatusCode(500, new { Message = "Query failed" });
    }
}
```

##### 3. Coupon Usage & Status Update

```csharp
[HttpPost("ApplyCoupon")]
public async Task<IActionResult> ApplyCoupon([FromBody] CouponApplicationRequestDto request)
{
    using var transaction = await _context.Database.BeginTransactionAsync();
    try
    {
        var coupon = await _context.Coupons
            .FirstOrDefaultAsync(c => c.CouponCode == request.CouponCode && 
                                    c.UserId == request.UserId);

        if (coupon == null)
        {
            return BadRequest(new { Message = "Coupon does not exist" });
        }

        if (coupon.IsUsed)
        {
            return BadRequest(new { Message = "Coupon has already been used" });
        }

        if (coupon.ExpiryDate < DateTime.Now)
        {
            return BadRequest(new { Message = "Coupon has expired" });
        }

        // Mark as used
        coupon.IsUsed = true;
        coupon.UsedTime = DateTime.Now;
        coupon.UsedOrderId = request.OrderId; // Associate with the order

        // Record usage history
        var usageHistory = new CouponUsageHistory
        {
            CouponId = coupon.Id,
            UserId = request.UserId,
            OrderId = request.OrderId,
            UsedTime = DateTime.Now,
            DiscountAmount = coupon.DiscountAmount,
            UsageSource = "LINE_BOT"
        };

        await _context.CouponUsageHistories.AddAsync(usageHistory);
        await _context.SaveChangesAsync();
        await transaction.CommitAsync();

        return Ok(new
        {
            Success = true,
            Message = "Coupon applied successfully",
            DiscountAmount = coupon.DiscountAmount
        });
    }
    catch (Exception ex)
    {
        await transaction.RollbackAsync();
        _logger.LogError(ex, "Failed to apply coupon: CouponCode={CouponCode}", request.CouponCode);
        return StatusCode(500, new { Message = "Application failed" });
    }
}
```

#### Technical Highlights

##### Multi-system Collaboration
- **Cross-system integration**: Seamless data integration between stamp collection and e-commerce systems
- **Flexible DataTable usage**: Processing dynamic stored procedure return results
- **LINE notification integration**: Real-time Flex Message push notifications upon successful redemption

##### Data Processing
- **Complex query optimization**: Using stored procedures to improve complex coupon query performance
- **Batch operation handling**: Transactional support for simultaneous multi-coupon redemption
- **Dynamic result conversion**: Flexible DataTable to JSON data format processing
- **Status tracking management**: Complete coupon usage status and timestamp records

##### Architecture Design
- **Standardized DTOs**: Unified request and response data structure design
- **Layered architecture**: Clear separation of business logic, data access, and external integration
- **Error recovery mechanism**: Data integrity protection under exceptional conditions
- **System integration capability**: Seamless integration with existing system architecture

---

## 5. CI/CD & Deployment Pipeline

To ensure the project is fully automated from code commit to deployment across environments while maintaining environment isolation, the project uses the following GitHub Actions pipelines:

### 5.1 Pipeline Overview

| Stage | Workflow File | Trigger Branch/Event | Target Environment | Core Actions |
| --- | --- | --- | --- | --- |
| **Build + Test** | `build-test.yml` | Any branch Push / PR | — | Restore → Build → Test |
| **Dev Deployment** | `build-test-publish-deploy_devTest.yml` | Push to `develop` branch | Self-hosted dev runner | Build → Test → Publish → Upload artifact → Deploy |
| **Main Branch Deployment** | `build-test-publish-deploy_merge.yml` | Merge to `main` branch | Self-hosted dev runner | Same as above |
| **Production Deployment** | `build-test-publish-deploy_release.yml` | GitHub Release published | Self-hosted prod runner | Same as above + Multi-retry mechanism |
| **Auto Rollback** | `rollback-deploy_release.yml` | GitHub Release deleted | Self-hosted prod runner | Fetch previous version → Auto rollback deploy |

### 5.2 Core Pipeline Snippets

The following excerpts highlight the key steps of each stage:

```yaml
# 1. build-test.yml: Basic build and test
- uses: actions/setup-dotnet@v2
  with:
    dotnet-version: '8.0.x'
- run: dotnet restore
- run: dotnet build --no-restore
- run: dotnet test --no-build --verbosity normal

# 2. build-test-publish-deploy_devTest.yml: Dev environment deployment
- name: Publish
  run: dotnet publish ProjectName.csproj -c Debug -o ${{env.DOTNET_ROOT}}/myapp
- name: Upload Artifact
  uses: actions/upload-artifact@v4
  with:
    name: .net-app
    path: ${{env.DOTNET_ROOT}}/myapp

# 3. Deploy Job (Self-hosted Runner)
- name: Take application offline
  run: New-Item -Type File -Name app_offline.htm -Path "C:\DeployPath\App" -Force
- name: Deploy to IIS
  run: Copy-Item ./* "C:\DeployPath\App" -Recurse -Force
- name: Bring application online
  run: Remove-Item "C:\DeployPath\App\app_offline.htm"
```

### 5.3 Technical Highlights

- **Multi-branch environment mapping**: `develop` → testing, `main` → integration testing, `release` → production, ensuring environment isolation and clear workflow.
- **High-availability deployment**: Automatic offline/online mechanism prevents users from encountering half-deployed states.
- **Retry & rollback mechanism**: Production releases include multi-retry and automatic rollback on Release deletion, greatly improving deployment stability.

---

## 6. Production Issue Resolution Experience

### 6.1 System Crash Investigation

**Symptom**: After running for some time, the entire API system would become unresponsive — even the Swagger documentation couldn't load, requiring an IIS restart to recover.

**Investigation Strategy**:
1. Analyzed code with multiple AI tools, checking for memory leaks, database connection issues, and other common problems
2. Repeatedly reviewed Program.cs configuration without finding obvious issues
3. Sought help from senior developers for in-depth code review

**Root Cause**:
The problem was traced to **duplicate `WebApplication.CreateBuilder` calls in Program.cs** causing configuration conflicts:

```csharp
// Problematic code: Builder created twice
Log.Logger = new LoggerConfiguration()
    .ReadFrom.Configuration(WebApplication.CreateBuilder(args).Configuration) // First instance
    .CreateLogger();

var builder = WebApplication.CreateBuilder(args); // Second instance
builder.Host.UseSerilog(); // Uses second builder, but config from the first
```

**Solution**:
- Unified to a single `WebApplication.CreateBuilder` instance
- Redesigned the Serilog configuration flow to avoid resource contention
- Implemented asynchronous log writing

### 6.2 LINE Webhook Integration Issue

**Symptom**: Webhook URL was correctly configured in the LINE Console, but status remained "inactive mode" and couldn't receive events.

**Attempted Solutions**:
1. **AI consultation**: Multiple AI tools provided standard configuration check suggestions
2. **Official channel**: LINE customer support responded that technical issues require self-investigation
3. **Tech community**: Searched Stack Overflow for related issues and actively posted for help

**Final Resolution**:
- Joined the **Facebook LINE Developer Community** and described the issue in detail
- With community members' assistance, identified a **server-side configuration conflict** through deep investigation
- After resolving the configuration conflict, Webhook was successfully activated

**Problem-solving Ability Demonstrated**:
- **Multi-channel approach**: Leveraged different resources (AI, official support, community) to solve technical issues
- **Community participation**: Actively participated in tech communities, building a developer network
- **Persistence**: Continued seeking solutions when facing complex problems without giving up

### 6.3 SQL Query Performance Optimization

**Background**: As user data grew, some complex queries began experiencing performance issues

**Optimization Strategy**:
- Used AI tools to analyze SQL execution plans and identify performance bottlenecks
- Created appropriate indexes for high-frequency queries
- Refactored complex query logic to reduce unnecessary JOIN operations
- Implemented stored procedures for complex business logic queries

**Results**: Multiple critical APIs saw response times improved from several seconds to the sub-100ms range

---

## 7. Professional Competencies & Highlights

- **LINE Messaging API mastery**: Event processing and signature verification
- **Complete OTP verification**: Full member binding flow design
- **Cross-team collaboration**: Close cooperation with design, planning, and testing teams with rapid iteration
- **Clear database design** & **uniqueness protection**: Led SSMS table design with consistency enforcement
- **EF Core Scaffold**: Rapid database structure mapping with encapsulated stored procedure calls
- **Flexible Flex Message management**: JSON templates that designers can modify directly
- **Modular service architecture**: Binding, OTP, Stamp, Redeem, RichMenu modules
- **Full CI/CD automation**: GitHub Actions deployment and rollback strategy
- **Production problem-solving ability**: Proactive learning and help-seeking for rapid system-level issue resolution

---

## 8. Project Results & Technical Value

### 8.1 Business Results
- **User scale**: Successfully supporting stable service for 24,000+ LINE users
- **System stability**: Battle-tested in production, with reliable error handling and recovery mechanisms
- **Business integration**: Complete integration of three independent systems for seamless user experience

### 8.2 Technical Growth
- **End-to-end development experience**: Independent development from zero to production
- **Problem-solving capability**: Analysis and resolution experience with complex technical issues
- **Collaboration skills**: Effective cross-functional team collaboration
- **Continuous learning mindset**: Leveraging AI tools and community resources for ongoing skill improvement

This project demonstrates the technical growth trajectory from beginner to independently developing a complete backend system, as well as the ability to solve complex problems in a real production environment.

---

*This portfolio demonstrates the complete technical capability and hands-on experience of independently developing a large-scale LINE Bot system*
