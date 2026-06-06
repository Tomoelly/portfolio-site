# Essay Contest Management System

> **Project Overview:** A social media referral essay submission system. Customers publish recommendation content on platforms such as Facebook, Instagram, and Google Reviews, then submit their posts through the system with URL proof. After passing a dual-layer review process, corresponding rewards are issued.
> 
> **Important Notice:** All code, API names, database structures, and other technical content in this portfolio have been fully desensitized and are presented solely for demonstrating technical capabilities. No proprietary company information is disclosed.

---

## Project Information

| Item | Details |
| --- | --- |
| **Project Type** | Enterprise-grade essay contest management system |
| **Role** | Backend Developer |
| **Tech Stack** | .NET 6 Web API, Entity Framework Core, SQL Server |
| **System Scale** | **13 core API endpoints, 11 status states, 3 submission plans** |
| **Architecture Highlights** | Multi-stage workflow control + Dual-layer review + **Dual LINE Bot architecture** |

### What is LINE?

LINE is the dominant messaging platform in Taiwan, Japan, and Southeast Asia (similar to WhatsApp or WeChat). With over 21 million monthly active users in Taiwan alone (90%+ penetration), LINE is the primary channel businesses use for customer communication. LINE provides a **Messaging API** that allows developers to build chatbots (LINE Bots) for automated customer interactions, push notifications, and rich interactive messages (Flex Messages).

---

## Business Problems Solved

### Core Challenges

- **Complex workflow control:** Design a complete submission lifecycle ensuring business process integrity
- **Duplicate submission prevention:** URL deduplication mechanism that considers submission status context
- **Dual Bot coordination:** Data synchronization and personalized push notifications between customer-facing Bot and consultant Bot
- **Secure file handling:** Multi-format file upload with security validation and size restrictions

### Solutions Delivered

- **Multi-plan submission support:** Facebook/Instagram, Google Reviews, PTT/Dcard platforms
- **Dual-layer review mechanism:** Consultant first review + Customer service final review for quality assurance
- **Complete business workflow:** Covering the entire submission lifecycle
- **Dual LINE Bot architecture:** Customer submission Bot + Consultant-dedicated push notification Bot
- **Personalized notification system:** Intelligent push notifications to assigned consultants

---

## System Design & User Experience

### End-to-End Product Design Process

This project was not just a backend implementation but a complete product development experience. I collaborated closely with UI/UX designers, from user journey design to final implementation, demonstrating full-stack thinking and cross-functional collaboration skills.

#### Business Flow Design
![figure-01-system-flow](https://hackmd.io/_uploads/S14nW5_Pge.png)

**Design Highlights:**
- **Multi-role collaboration workflow:** Complete workflows for customers, consultants, and customer service
- **Visualized state transitions:** Clearly defined submission process flow
- **Edge case handling:** Rejection, resubmission, and other boundary scenarios

#### Customer Submission Experience
![figure-02-client-submit](https://hackmd.io/_uploads/H1OlU5Owxe.png)

**Technical Implementation Highlights:**
- **LINE Bot interface integration:** Seamless social media submission experience
- **Plan-specific dynamic UI:** Differentiated form validation for Plans A/B/C
- **File upload optimization:** 5MB limit with format validation and frontend feedback

#### Customer Edit Functionality
![figure-03-client-edit](https://hackmd.io/_uploads/B1iW89uPee.png)

**Complex Edit Logic Design:**
- **Dynamic plan switching:** Supports free conversion between Plans A/B/C; backend automatically handles data structure differences
- **Edit permission control:** Editing allowed only at specific stages to ensure review process integrity
- **Data retention mechanism:** Intelligently preserves compatible field data during plan switches, reducing redundant user input
- **File management optimization:** Complex file management logic supporting image retain/delete/add operations
- **Real-time validation feedback:** Instant form validation and error prompts during editing

#### Consultant Review System
![figure-04-consultant-review](https://hackmd.io/_uploads/Bk2MU5_Deg.png)

**Backend Features:**
- **Personalized push notification API:** Precise delivery to assigned consultants
- **Review status management:** State machine logic implementation
- **Review workflow control:** Complete submission review workflow

#### Consultant Management Overview
![figure-05-consultant-overview](https://hackmd.io/_uploads/HkK7L9dwxg.png)

**Management Features:**
- **Permission scope control:** Consultants can only view submissions from their assigned customers
- **Review progress categorization:** Intelligent classification into Pending, Awaiting CS, Completed, and Withdrawn
- **Efficient query design:** Data scope restriction and performance optimization based on reviewer identity
- **Comprehensive information display:** Submission time, customer info, plan type, review status at a glance

#### LINE Bot Message Design
<img src="https://hackmd.io/_uploads/H1W48cOPex.png" alt="figure-06-line-messages" width="40%">

**Integration Experience:**
- **Flex Message API:** Rich card-style message design
- **Progress notification system:** Intelligent push notifications based on business logic
- **Bidirectional interaction:** Complete feedback loop between customer responses and system replies

### Cross-Functional Collaboration Value

**Frontend-Backend Coordination:**
- API interface design based on design mockups, ensuring smooth frontend experience
- Data structure optimization supporting complex UI state management requirements
- Error handling mechanisms providing user-friendly feedback

**Product Thinking:**
- Not just writing code, but understanding business logic and user needs
- Participating in product design discussions, providing feasibility input from a technical perspective
- Prioritizing user experience by considering frontend implementation efficiency in backend design

---

## System Architecture

### Core Architecture

**Three-Layer Design:**
- **Dual LINE Bot Layer** <-> **Web API (.NET 6)** <-> **SQL Server (CompanyDigital)**

**Layer Responsibilities:**
- **Dual LINE Bot Layer:** Customer submission Bot, Consultant push notification Bot, real-time notifications
- **API Service Layer:** Business logic, personalized push notifications, file processing
- **Database Layer:** Submission data, consultant assignments, status definitions

### Database Schema Design

**Integration design built on existing database:**

- **Activity master:** Submission data, workflow control, plan configuration
- **Content management:** Image files, social media links, attachment processing
- **Status system:** Complete status definitions, transition rules, permission control
- **Review history:** Complete operation records, timestamps, accountability tracking
- **User management:** LINE user data, consultant assignments, permission groups
- **Notification events:** Event-driven push notifications, template management, delivery records

**Technical Highlights:**

- EF Core Database-First approach for database integration
- Rapid entity model generation via Scaffold commands
- Support for complex one-to-many and many-to-many relationship queries
- Cross-table transaction handling with data consistency guarantees

## EF Core Context Summary

Based on CompanyDigitalContext, core DbSets and relationships:

| Entity | Purpose | Relationships |
| --- | --- | --- |
| **LineAtPost101** | Submission master record, stores core data for each essay submission | 1:N <-> LineAtPost101Image, 1:N <-> LineAtPost101Link, N:1 <-> LineAtPost101Plan, N:1 <-> LineAtPost101Status, N:1 <-> LineAtProfile |
| **LineAtPost101Image** | Image list uploaded for Plan A | N:1 <-> LineAtPost101 |
| **LineAtPost101Link** | Submission platform links (Google Review, Social Media, etc.) | N:1 <-> LineAtPost101 |
| **LineAtPost101Plan** | Plan definitions (A, B, C) | 1:N <-> LineAtPost101 |
| **LineAtPost101Status** | Status definitions (11 states, with CanEdit/CanReview fields) | 1:N <-> LineAtPost101 |
| **LineAtPost101StatusHistory** | Status change history log | N:1 <-> LineAtPost101, N:1 <-> LineAtPost101Status |
| **LineAtProfile** | LINE user profile data | 1:N <-> LineAtPost101 |
| **LineAtEvent** | Collects and logs LINE Webhook events (message, follow/unfollow, postback, etc.) for downstream workflow triggers and debugging | N:1 <-> LineAtProfile (event source user) |

---

## System Configuration

Core configurations in `Program.cs` (sensitive details such as secret keys and connection strings are managed via environment variables and not disclosed here):

- **Structured Logging**
    
    Serilog integration with the host pipeline, supporting dual output to file and console with unified log format.
    
- **Database Connection with Retry**
    
    `AddDbContext<CompanyDigitalContext>` with SQL Server provider reading the `CompanyDigital` connection string, with retry-on-failure strategy enabled.
    
- **JWT Bearer Authentication**
    
    JWT Bearer authentication enabled with authorization rules protecting all `/api/post101` endpoints. Token signing keys and parameters are injected via environment variables.
    
- **CORS Policy**
    
    Required origins opened based on frontend domain requirements, with restricted HTTP methods and headers.
    
- **HttpClient DI**
    
    Named and Typed HttpClient instances injected for `FlexMessageService`, `LineAuthService`, and other third-party API calls.
    

```csharp
// Example (sensitive settings omitted):
builder.Services.AddDbContext<CompanyDigitalContext>(opt =>
    opt.UseSqlServer(
        builder.Configuration.GetConnectionString("CompanyDigital"),
        sql => sql.EnableRetryOnFailure()
    ));

builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options =>
    {
        // Token parameters and signing keys loaded from environment variables
    });

builder.Services.AddCors(options => { /* Specified frontend origins */ });
builder.Services.AddHttpClient("FlexMessageApi", client => { /*...*/ });
builder.Services.AddHttpClient<LineAuthService>();
```

---

## Core Feature Modules (Based on Actual Controller)

### 1. URL Duplicate Submission Prevention System

**Business Context:** In the essay submission system, the same Google Review or social media link should not be submitted multiple times. However, this restriction must consider the current status of existing submissions. For example, a withdrawn submission's link should be reusable, but a link under active review should not allow duplicate submissions.

**Technical Design:** I implemented an intelligent duplicate-checking mechanism -- not simple URL deduplication, but dynamic checking based on status permissions. The system checks the status of existing submissions with the same URL, blocking new submissions only when the existing submission is in a "non-resubmittable" state.

**Value Delivered:** This design protects business rule integrity while maintaining a user-friendly experience, avoiding overly strict restrictions that would cause usability issues.

**Technical Highlights:** Business logic validation, duplicate submission prevention, status-driven permission control

```csharp
/// <summary>
/// URL duplicate submission check - demonstrates business logic validation design.
/// Core concept: not simple URL deduplication, but intelligent checking based on
/// submission status. E.g., withdrawn submissions' URLs can be reused, but those
/// under review cannot.
/// </summary>
[HttpPost("CheckUrlSubmitted")]
public async Task<ActionResult<APIResultT<bool>>> CheckUrlSubmitted(
    [FromBody] CheckUrlRequestDto request)
{
    try
    {
        // 1. URL normalization - prevent duplicate detection errors due to
        //    casing or whitespace (e.g., "HTTP://ABC.com " vs "http://abc.com")
        var normalizedUrl = request.Url?.Trim().ToLowerInvariant();
        
        // 2. Intelligent duplicate check - dynamic validation based on submission status.
        //    Business logic: withdrawn submission URLs are reusable; URLs under review are not.
        var existingSubmission = await _context.SubmissionPosts
            .Include(p => p.Status)    // Load status info (includes CanUrlReSubmit permission)
            .Include(p => p.Links)     // Load link info
            .Where(p => 
                // Condition 1: Find submissions with the same URL
                p.Links.Any(l => l.Url.ToLower() == normalizedUrl) &&
                // Condition 2: Submission is in a non-resubmittable state (e.g., under review, approved)
                p.Status.CanUrlReSubmit == false) // Status permission control - core design
            .FirstOrDefaultAsync();
            
        // 3. Result determination and response
        bool isSubmitted = existingSubmission != null;
        
        // Unified API response format with success flag, data, and message
        return Ok(new APIResultT<bool>
        {
            IsSuccess = true,
            Data = isSubmitted,  // true = cannot submit, false = can submit
            Message = isSubmitted ? "This URL has been submitted and is under review" 
                                 : "URL is available for submission"
        });
    }
    catch (Exception ex)
    {
        // Error handling: log detailed errors for debugging, return concise user message
        _logger.LogError(ex, "URL duplicate check failed");
        return StatusCode(500, "Check failed");
    }
}
```

### 2. Submission & Management System

**Business Requirement:** The system needs to support three different submission plans, each with distinct data structures and validation rules. Plan A handles image uploads, Google Review links, and social media links; Plan B requires only a Facebook group link; Plan C handles PTT or Dcard forum links.

**Technical Challenges & Solutions:** Facing complex business logic, I adopted a strategy-pattern approach for handling plan-specific logic, while implementing comprehensive file security validation including file size limits, quantity limits, and format verification.

**Technical Highlights:** Multi-plan support, file processing, cross-database transactions

#### Submission Core Logic (Based on Actual Implementation)

```csharp
/// <summary>
/// Customer submission - demonstrates complex business logic handling.
/// Core: handles multi-scenario submission logic including JWT authentication,
/// file security validation, plan-specific processing, database transactions,
/// and notification delivery.
/// </summary>
[Authorize(AuthenticationSchemes = "BearerPost101", Roles = "lineUser")]
[HttpPost("SubmitPost")]
public async Task<ActionResult<APIResultT<object>>> SubmitPost(
    [FromForm] SubmitPostRequestDto request,        // Submission data
    [FromForm] List<IFormFile> instagramStories)    // Image files (Plan A only)
{
    try
    {
        // Step 1: JWT authentication and parsing
        // Extract user ID from JWT token to ensure submission is linked to the correct user.
        // Claims-based authentication prevents identity spoofing.
        var lineUserId = GetLineUserIdFromJwt();
        if (string.IsNullOrEmpty(lineUserId))
        {
            // Unified error response: MessageCode enables frontend i18n or special handling
            response.MessageCode = 4001;
            response.Message = "Cannot parse Line User ID from JWT. Please verify authorization.";
            return Unauthorized(response);
        }

        // Step 2: Multi-layer file security validation
        // File upload security and system resource protection
        // 2.1 Quantity limit - prevent system overload from excessive uploads
        if (instagramStories.Count > 3)
        {
            response.MessageCode = 4007;
            response.Message = "Maximum of 3 images allowed";
            return BadRequest(response);
        }

        // 2.2 File size limit - prevent large-file attacks and control storage costs
        const long maxFileSize = 5 * 1024 * 1024; // 5MB
        foreach (var file in instagramStories)
        {
            if (file.Length > maxFileSize)
            {
                // Per-file error message so the user knows which file is problematic
                response.MessageCode = 4008;
                response.Message = $"Each image must be under 5MB. Exceeded: {file.FileName}";
                return BadRequest(response);
            }
        }

        // Step 3: User profile lookup or creation
        // Query-first-then-create strategy avoids duplicate profiles, maintaining data consistency
        var profile = await _context.UserProfiles
            .FirstOrDefaultAsync(p => p.UserId == lineUserId);

        if (profile == null)
        {
            // First-time submitter: auto-create profile record
            // Benefit: reduces registration steps, improves user experience
            profile = new UserProfile
            {
                UserId = lineUserId,
                CreateDate = DateTime.Now
            };
            _context.UserProfiles.Add(profile);
        }

        // Step 4: Submission plan validation
        // Verify plan validity to ensure the submitted plan is system-supported
        var plan = await _context.SubmissionPlans
            .FirstOrDefaultAsync(p => p.PlanName == request.PlanName);

        if (plan == null)
        {
            return BadRequest("Invalid plan name");
        }

        // Step 5: Submission master record creation
        // Initialize core data structure and status settings
        var post = new SubmissionPost
        {
            ProfileId = profile.ProfileId,
            PlanId = plan.PlanId,
            StatusId = 1,               // Default: pending first review
            StatusDate = DateTime.Now,
            UploadTime = DateTime.Now,
            IsWithdrawn = false
        };
        _context.SubmissionPosts.Add(post);

        // Step 6: Plan-specific processing - strategy pattern design
        // Different plans have different data structures and validation rules.
        // Plan A: images + Google Review + social media
        // Plan B: Facebook group only
        // Plan C: PTT/Dcard forum only
        switch (request.PlanName)
        {
            case "A": // Most complex: handles image uploads + multiple link types
                await HandlePlanASubmission(request, instagramStories, post);
                break;
            case "B": // Medium complexity: Facebook link only
                await HandlePlanBSubmission(request, post);
                break;
            case "C": // Simplest: forum link only
                await HandlePlanCSubmission(request, post);
                break;
        }

        // Step 7: Status history logging - complete audit trail design
        // Business requirement: track every status change for full auditability
        post.SubmissionStatusHistories.Add(new SubmissionStatusHistory
        {
            Post = post,
            StatusId = 1,              // Initial status
            UpdateDate = DateTime.Now
        });

        // Step 8: Database commit - ensure all data is saved atomically
        await _context.SaveChangesAsync();

        // Step 9: Real-time notification - event-driven design
        // Enhances UX: immediately notify the user of submission status
        await SendSubmissionNotifications(lineUserId, request.PlanName, post.PostId);

        return Ok(new { PostId = post.PostId, Message = "Submission successful" });
    }
    catch (Exception ex)
    {
        // Global error handling: log detailed errors for debugging, return concise user message
        _logger.LogError(ex, "Submission failed");
        return StatusCode(500, "System error");
    }
}

/// <summary>
/// Plan A processing logic - the most complex submission plan.
/// Core: unified management of multiple data types (text, links, files).
/// </summary>
private async Task HandlePlanASubmission(SubmissionRequestDto request,
    List<IFormFile> attachments, SubmissionPost post)
{
    // 1. Business rule validation - Plan A completeness check
    // Plan A requires: Google Review, social media link, and Instagram story images
    if (string.IsNullOrEmpty(request.GoogleReviewUrl) ||
        string.IsNullOrEmpty(request.SocialMediaUrl) ||
        !attachments.Any())
    {
        throw new ArgumentException("Plan A required fields are missing");
    }

    // 2. Batch link processing - demonstrates efficient bulk operations
    // Using AddRange for batch database insertion, improving performance
    _context.SubmissionLinks.AddRange(new[]
    {
        new SubmissionLink
        {
            Post = post,
            Url = request.GoogleReviewUrl,    // Google Review link
            LinkType = "GoogleReview"
        },
        new SubmissionLink
        {
            Post = post,
            Url = request.SocialMediaUrl,     // Facebook/Instagram link
            LinkType = "SocialMedia"
        }
    });

    // 3. Image file processing and storage
    foreach (var file in attachments)
    {
        // 3.1 Secure filename handling
        // GUID prevents filename collisions while preserving the original extension
        var fileName = $"{Guid.NewGuid()}{Path.GetExtension(file.FileName)}";
        var savePath = Path.Combine(_fileStoragePath, fileName);

        // 3.2 File persistence
        // Using 'using' ensures proper stream resource disposal, preventing memory leaks
        using (var stream = new FileStream(savePath, FileMode.Create))
        {
            await file.CopyToAsync(stream);
        }

        // 3.3 Relative path handling - cross-platform compatibility
        // Unified forward slashes for consistency across Windows and Linux
        var relativePath = Path.Combine(_relativePath, fileName).Replace("\\", "/");

        // 3.4 Image record creation
        _context.SubmissionImages.Add(new SubmissionImage
        {
            Post = post,
            ImageUrl = relativePath   // Relative path for easy future migration
        });
    }
}
```

### 3. Complex Status Management System

**Business Complexity:** The essay system manages the entire submission lifecycle from initial submission to final completion, encompassing 11 distinct statuses. Each status has explicit permission definitions determining what operations users can perform (edit, withdraw) and what review actions reviewers can take.

**Design Philosophy:** I designed a flexible state machine system where each status includes boolean fields such as CanEdit, CanReview, and CanReject to control operation permissions. This design enables dynamic business process control while maintaining good extensibility.

**Technical Value:** This status management system not only solves complex workflow control problems but, more importantly, demonstrates my ability to abstract and implement business logic.

**Technical Highlights:** Status transition validation, complete audit trail, permission control

#### Submission Edit & Status Update

```csharp
/// <summary>
/// Submission edit update - demonstrates complex business logic handling.
/// Core: complete edit workflow supporting plan switching, file management,
/// and status control.
/// Key: achieving user-friendly plan switching while maintaining data consistency.
/// </summary>
[Authorize(AuthenticationSchemes = "BearerPost101", Roles = "lineUser")]
[HttpPut("UpdatePost")]
public async Task<ActionResult<APIResultT<string>>> UpdatePost(
    [FromForm] UpdateRequestDto request,
    [FromForm] List<IFormFile> instagramStories)
{
    // Database transaction - ensures atomicity for complex operations
    // Involves changes across multiple tables (submission, links, images, status history)
    using var transaction = await _context.Database.BeginTransactionAsync();
    try
    {
        // Step 1: Eager-load all related data
        // EF Core Include to load all associations in one query, avoiding N+1 problems
        var post = await _context.LineAtPost101s
            .Include(p => p.LineAtPost101Images)    // Existing images (may delete/retain)
            .Include(p => p.LineAtPost101Links)     // Existing links (may update)
            .Include(p => p.Status)                 // Status info (check edit permission)
            .Include(p => p.Plan)                   // Current plan (may switch)
            .FirstOrDefaultAsync(p => p.Post101Id == request.Post101Id);

        // Step 2: State machine permission verification
        // Check if the current status allows editing; submissions under review
        // or already completed cannot be edited
        if (post?.Status?.CanEdit != true)
        {
            return BadRequest("Current status does not allow editing");
        }

        // Step 3: Plan change handling
        // Supports plan switching during edit (e.g., A -> B);
        // must handle data structure differences between plans
        var targetPlan = await _context.SubmissionPlans
            .FirstOrDefaultAsync(p => p.PlanName == request.PlanName);

        // Step 4: Plan-specific update logic
        // Different plans require different update strategies:
        // intelligently retain compatible data, clear incompatible data
        switch (request.PlanName)
        {
            case "A": // Images + multiple links: most complex update logic
                await UpdatePlanAContent(request, instagramStories, post);
                break;
            case "B": // Facebook link only: must clear images and Google Review
                await UpdatePlanBContent(request, post);
                break;
            case "C": // Forum link only: must clear images and other links
                await UpdatePlanCContent(request, post);
                break;
        }

        // Step 5: Submission status reset
        // After content edit, re-enter review process to ensure
        // modified content goes through complete review
        post.PlanId = targetPlan.PlanId;
        post.StatusId = 1;                  // Back to pending review
        post.StatusDate = DateTime.Now;
        post.UploadTime = DateTime.Now;

        // Step 6: Status history logging
        // Complete audit trail recording every status change with timestamp
        post.SubmissionStatusHistories.Add(new SubmissionStatusHistory
        {
            Post = post,
            StatusId = 1,                   // Back to pending review
            UpdateDate = DateTime.Now
        });

        // Step 7: Transaction commit
        await _context.SaveChangesAsync();
        await transaction.CommitAsync();

        return Ok(new { Message = "Update successful" });
    }
    catch (Exception ex)
    {
        // Error handling: transaction rollback ensures data consistency
        await transaction.RollbackAsync();
        _logger.LogError(ex, "Submission update failed");
        throw;
    }
}

/// <summary>
/// Plan A content update - image retention and addition logic.
/// </summary>
private async Task UpdatePlanAContent(UpdateRequestDto request,
    List<IFormFile> instagramStories, LineAtPost101 post)
{
    // Image management: retain specified images, delete others
    var retainImageIds = request.KeepIgImageIds ?? new List<int>();
    var imagesToRemove = post.SubmissionImages
        .Where(img => !retainImageIds.Contains(img.ImageId))
        .ToList();

    _context.SubmissionImages.RemoveRange(imagesToRemove);

    // New image processing
    if (instagramStories?.Any() == true)
    {
        foreach (var file in instagramStories)
        {
            var fileName = $"{Guid.NewGuid()}_{Path.GetFileName(file.FileName)}";
            var savePath = Path.Combine(_instagramStoriesFolderPath, fileName);

            using (var stream = new FileStream(savePath, FileMode.Create))
                await file.CopyToAsync(stream);

            var relativePath = Path.Combine(_relativePath, fileName).Replace("\\", "/");
            _context.SubmissionImages.Add(new SubmissionImage
            {
                Post = post,
                ImageUrl = relativePath
            });
        }
    }

    // Link update: intelligent upsert mechanism
    UpdateOrCreateLink(post, request.GoogleReviewUrl, "GoogleReview");
    UpdateOrCreateLink(post, request.FbOrIgUrl, "FacebookOrInstagram");
}

/// <summary>
/// Intelligent link upsert - avoids creating duplicate links of the same type.
/// </summary>
private void UpdateOrCreateLink(SubmissionPost post, string url, string linkType)
{
    var existingLink = post.SubmissionLinks
        .FirstOrDefault(l => l.LinkType == linkType);

    if (existingLink != null)
    {
        existingLink.Url = url; // Update existing link
    }
    else
    {
        _context.SubmissionLinks.Add(new SubmissionLink
        {
            PostId = post.PostId,
            Url = url,
            LinkType = linkType
        });
    }
}
```

### 4. Dual-Layer Review Workflow System

**Business Requirement:** To ensure submission content quality and compliance, the system implements a dual-layer review mechanism. Consultants handle the first review, checking link validity and image quality; customer service performs the final review for ultimate quality assurance. Each review layer can assess individual items (links, images) independently.

**Implementation Challenge:** The biggest challenge was designing flexible review logic that supports partial approval and partial rejection of complex review results. I designed per-item review status tracking, with the overall submission's next status determined by the aggregate result of all individual item reviews.

**System Value:** This review workflow not only improves content quality but, more importantly, establishes a complete accountability trail -- every review decision has a clear operator and timestamp record.

**Technical Highlights:** Consultant first review + CS final review, per-item review, status-driven notifications

#### Consultant Review Process

```csharp
/// <summary>
/// Consultant review submission - demonstrates complex review logic design.
/// Supports per-item review with overall result determination.
/// </summary>
[Authorize(AuthenticationSchemes = "JwtBearer", Roles = "reviewer")]
[HttpPut("review")]
public async Task<ActionResult<APIResultT<string>>> SubmitReview(
    [FromBody] ReviewRequestDto request)
{
    try
    {
        // 1. Load submission and all related data
        var post = await _context.SubmissionPosts
            .Include(p => p.SubmissionLinks)
            .Include(p => p.SubmissionImages)
            .Include(p => p.Status)
            .Include(p => p.Plan)
            .Include(p => p.Profile)
            .FirstOrDefaultAsync(p => p.PostId == request.PostId);

        if (post?.Status?.CanReview != true)
        {
            return BadRequest("Current status does not allow review");
        }

        // 2. Per-link review processing
        foreach (var linkReview in request.LinkReviews)
        {
            var link = post.SubmissionLinks
                .FirstOrDefault(l => l.LinkId == linkReview.LinkId);

            if (link == null)
            {
                return BadRequest($"Invalid link ID: {linkReview.LinkId}");
            }

            link.IsApprovedByReviewer = linkReview.IsApproved;
            link.ReviewDate = DateTime.Now;
        }

        // 3. Image review processing (Plan A only)
        if (post.Plan.PlanName == "A")
        {
            if (!request.IsImageApproved.HasValue)
            {
                return BadRequest("Plan A requires image review result");
            }

            post.IsImageApprovedByReviewer = request.IsImageApproved.Value;
            post.ReviewDate = DateTime.Now;
        }

        // 4. Overall review result determination - business logic design
        bool overallApproved = request.LinkReviews.All(r => r.IsApproved == true) &&
                              (request.IsImageApproved ?? true);

        post.StatusId = overallApproved ? 7 : 6; // Approved -> pending final review; Rejected -> first review failed
        post.StatusDate = DateTime.Now;

        // 5. Review history logging
        post.SubmissionStatusHistories.Add(new SubmissionStatusHistory
        {
            PostId = post.PostId,
            StatusId = post.StatusId,
            UpdateDate = DateTime.Now
        });

        await _context.SaveChangesAsync();

        // 6. Status-driven notification delivery
        await SendReviewNotification(post, overallApproved);

        string reviewResult = overallApproved ? "approved" : "rejected";
        return Ok(new { Message = $"Review result '{reviewResult}' submitted" });
    }
    catch (Exception ex)
    {
        _logger.LogError(ex, "Consultant review failed");
        return StatusCode(500, "Review processing error");
    }
}

/// <summary>
/// Status-driven notification delivery - event-driven design.
/// </summary>
private async Task SendReviewNotification(SubmissionPost post, bool isApproved)
{
    var notificationApiUrl = _configuration["ExternalService:NotificationAPI"];
    var templateType = isApproved ? "ApprovalNotification" : "RejectionNotification";

    var notificationData = new
    {
        UserId = post.Profile.UserId,
        UserName = post.Profile.RealName,
        PlanName = post.Plan.PlanName,
        TemplateType = templateType
    };

    var content = new StringContent(
        JsonConvert.SerializeObject(notificationData),
        Encoding.UTF8,
        "application/json");

    var response = await _httpClient.PostAsync(notificationApiUrl, content);

    if (!response.IsSuccessStatusCode)
    {
        _logger.LogError("Notification delivery failed: {StatusCode}", response.StatusCode);
    }
}
```

### 5. Consultant Management Overview System

**Technical Highlights:** Permission control, data grouping, optimized query performance

```csharp
/// <summary>
/// Reviewer management overview - demonstrates complex queries and permission control.
/// Data scope restriction based on reviewer identity.
/// </summary>
[Authorize(AuthenticationSchemes = "JwtBearer", Roles = "reviewer")]
[HttpPost("overview")]
public async Task<ActionResult<APIResultT<List<ReviewOverviewResponseDto>>>>
    GetReviewOverview([FromBody] ReviewOverviewRequestDto request)
{
    try
    {
        // 1. Reviewer identity verification and permission scope
        var userId = GetUserIdFromJwt();
        var reviewerAccount = await _context.ReviewerProfiles
            .Where(emp => emp.UserId == userId)
            .Select(emp => emp.AccountId)
            .FirstOrDefaultAsync();

        if (reviewerAccount == null)
        {
            return NotFound("Reviewer profile not found");
        }

        // 2. Status group definitions - business logic abstraction
        var statusGroups = new Dictionary<int, int[]>
        {
            { 1, new[] { 1, 4, 10 } },    // Pending
            { 2, new[] { 7, 8 } },        // Awaiting CS
            { 3, new[] { 6, 9, 11, 12 } }, // Completed
            { 4, new[] { 3, 5 } }         // Withdrawn
        };

        var targetStatuses = statusGroups.GetValueOrDefault(request.StatusGroup,
                                                           Array.Empty<int>());

        // 3. Query customers within permission scope
        var clientProfileIds = await _context.UserProfiles
            .Where(profile => profile.AssignedReviewerId == reviewerAccount)
            .Select(profile => profile.ProfileId)
            .ToListAsync();

        // 4. Submission query with projection - optimized query performance
        var posts = await _context.SubmissionPosts
            .Where(post => clientProfileIds.Contains(post.ProfileId) &&
                          targetStatuses.Contains(post.StatusId))
            .OrderByDescending(post => post.UploadTime) // Most recent first
            .Select(post => new ReviewOverviewResponseDto
            {
                PostId = post.PostId,
                UploadTime = post.UploadTime,
                CustomerName = post.Profile.RealName,
                CustomerPhone = post.Profile.Phone,
                PlanName = post.Plan.PlanName,
                StatusId = post.StatusId,
                StatusDate = post.StatusDate,
                ReviewStatus = post.Status.ReviewStatus
            })
            .ToListAsync();

        return Ok(new
        {
            Data = posts,
            Message = $"Found {posts.Count} records"
        });
    }
    catch (Exception ex)
    {
        _logger.LogError(ex, "Reviewer overview query failed");
        return StatusCode(500, "Query failed");
    }
}
```

### 6. Submission Withdrawal & Status Management

**Technical Highlights:** User self-service operations, status transition control, permission verification

```csharp
/// <summary>
/// Submission withdrawal - demonstrates user self-service operation design.
/// Supports status validation and complete operation logging.
/// </summary>
[Authorize(AuthenticationSchemes = "JwtBearer", Roles = "user")]
[HttpPut("WithdrawPost")]
public async Task<ActionResult<APIResultT<string>>> WithdrawPost(
    [FromBody] WithdrawPostRequestDto request)
{
    try
    {
        var lineUserId = GetLineUserIdFromJwt();
        
        // Submission and permission verification
        var post = await _context.SubmissionPosts
            .Include(p => p.Status)
            .Include(p => p.Profile)
            .FirstOrDefaultAsync(p => p.PostId == request.PostId &&
                                    p.Profile.UserId == lineUserId);
                                    
        if (post == null)
        {
            return NotFound("Submission not found");
        }
        
        // Status permission check - only specific statuses allow withdrawal
        if (!CanWithdraw(post.StatusId))
        {
            return BadRequest("Current status does not allow withdrawal");
        }
        
        // Execute withdrawal
        post.StatusId = 5; // Withdrawn status
        post.StatusDate = DateTime.Now;
        post.IsWithdrawn = true;
        
        // Withdrawal history logging
        post.StatusHistories.Add(new StatusHistory
        {
            Post = post,
            StatusId = 5,
            UpdateDate = DateTime.Now,
            OperatorType = "User"
        });
        
        await _context.SaveChangesAsync();
        
        // Withdrawal notification
        await SendWithdrawNotification(lineUserId, post.PostId);
        
        return Ok(new { Message = "Submission withdrawn successfully" });
    }
    catch (Exception ex)
    {
        _logger.LogError(ex, "Submission withdrawal failed");
        return StatusCode(500, "Withdrawal operation failed");
    }
}

/// <summary>
/// Withdrawal permission check - business logic encapsulation.
/// </summary>
private bool CanWithdraw(int statusId)
{
    // Define withdrawable status IDs (pending review, etc.)
    var withdrawableStatuses = new[] { 1, 4, 7, 10 };
    return withdrawableStatuses.Contains(statusId);
}
```

### 7. Rejection Reason Query System

**Technical Highlights:** Multi-level error information, relational data queries, UX optimization

```csharp
/// <summary>
/// Rejection reason query - demonstrates multi-level error information design.
/// Provides complete review failure reasons with improvement suggestions.
/// </summary>
[Authorize(AuthenticationSchemes = "JwtBearer", Roles = "user,reviewer")]
[HttpPost("GetRejectionReason")]
public async Task<ActionResult<APIResultT<GetRejectionReasonResponseDto>>> 
    GetRejectionReason([FromBody] PostDetailRequestDto request)
{
    try
    {
        // Multi-level relational query - demonstrates advanced EF Core usage
        var rejectionData = await _context.SubmissionPosts
            .Include(p => p.Links)
            .Where(p => p.PostId == request.PostId)
            .Select(p => new GetRejectionReasonResponseDto
            {
                // Image review rejection reason
                ImageRejectionReason = p.ImageRejectionReasonCs,
                
                // Per-link review rejection reasons
                Links = p.Links.Select(link => new LinkRejectionReasonDto
                {
                    LinkType = link.LinkType,
                    RejectionReason = link.RejectionReasonCs,
                    Url = link.Url,
                    SuggestedAction = GetSuggestedAction(link.RejectionReasonCs)
                }).ToList()
            })
            .FirstOrDefaultAsync();
            
        if (rejectionData == null)
        {
            return NotFound("Submission not found");
        }
        
        return Ok(new APIResultT<GetRejectionReasonResponseDto>
        {
            IsSuccess = true,
            Data = rejectionData,
            Message = "Rejection reasons retrieved successfully"
        });
    }
    catch (Exception ex)
    {
        _logger.LogError(ex, "Rejection reason query failed");
        return StatusCode(500, "Query failed");
    }
}

/// <summary>
/// Improvement suggestion generation - demonstrates UX-oriented design thinking.
/// </summary>
private string GetSuggestedAction(string rejectionReason)
{
    return rejectionReason switch
    {
        "Invalid URL" => "Please verify the URL format is correct and accessible",
        "Content does not meet requirements" => "Please check if the content complies with campaign rules",
        "Poor image quality" => "Please re-upload a higher-resolution image",
        _ => "Please revise based on review feedback and resubmit"
    };
}
```

### 8. Reward Collection Confirmation System

**Technical Highlights:** Complete business lifecycle closure, status integrity, user experience

```csharp
/// <summary>
/// Reward collection confirmation - demonstrates complete business lifecycle design.
/// Ensures submission process integrity and user satisfaction tracking.
/// </summary>
[Authorize(AuthenticationSchemes = "JwtBearer", Roles = "user,visitor")]
[HttpPut("ConfirmCollect")]
public async Task<ActionResult<APIResultT<string>>> ConfirmCollect(
    [FromBody] PostDetailRequestDto request)
{
    try
    {
        var post = await _context.SubmissionPosts
            .Include(p => p.Status)
            .FirstOrDefaultAsync(p => p.PostId == request.PostId);
            
        if (post == null)
        {
            return NotFound("Submission not found");
        }
        
        // Status validation - only approved submissions can be collected
        if (post.StatusId != 8) // Final review passed
        {
            return BadRequest("This submission has not been approved or has already been collected");
        }
        
        // Update to collected status
        post.StatusId = 13; // Collection completed
        post.StatusDate = DateTime.Now;
        post.CollectDate = DateTime.Now;
        
        // Collection history logging
        post.StatusHistories.Add(new StatusHistory
        {
            Post = post,
            StatusId = 13,
            UpdateDate = DateTime.Now,
            OperatorType = "User"
        });
        
        await _context.SaveChangesAsync();
        
        // Completion notification and satisfaction survey trigger
        await SendCompletionSurvey(post.Profile.UserId, post.PostId);
        
        return Ok(new { Message = "Collection confirmed. Thank you for participating!" });
    }
    catch (Exception ex)
    {
        _logger.LogError(ex, "Collection confirmation failed");
        return StatusCode(500, "Confirmation failed");
    }
}
```

---

## CI/CD Deployment Architecture

**Hands-On Experience:** Participated in the full multi-environment CI/CD deployment architecture implementation within the team, gaining practical understanding of deployment pipeline design principles and operations.

**Technical Growth:** Through participating in CI/CD setup and maintenance, expanded my technical perspective from pure backend development to the complete DevOps process, building practical full-stack deployment capabilities.

### Architecture Design
```
Feature Branch -> build-test.yml (code quality gate)
     |
Deploy Branch -> build-test-publish-deploy_devTest.yml (test environment)
     |  
Main Branch -> build-test-publish-deploy_merge.yml (development environment)
     |
Release -> build-test-publish-deploy_release.yml (production environment)
```

### Core Technical Features
- **Self-hosted Runners:** Separate deployment for dev/test and production environments
- **Triple retry mechanism:** Auto-retry up to 3 times on deployment failure for reliability
- **Graceful shutdown:** `app_offline.htm` ensures existing requests complete before deployment
- **Full IIS integration:** Application Pool management, WAS service management, IIS reset
- **Intelligent rollback:** Release deletion automatically triggers previous version rollback
- **Environment-specific strategies:** Full management for dev, stability-focused for production

### Deployment Workflow Design

```yaml
# Production environment deployment core workflow
name: Production Deploy
on:
  release:
    types: [published]

jobs:
  deploy-production:
    runs-on: [self-hosted, production-server]
    steps:
      # 1. Graceful shutdown
      - name: Take application offline
        run: New-Item -Type File -Name app_offline.htm -Path C:\WebServer\CompanyAPI -Force

      - name: Sleep for graceful shutdown
        run: Start-Sleep -s 5
        shell: powershell

      # 2. Triple retry deployment mechanism
      - name: Deploy to IIS (first attempt)
        id: deploy-first-attempt
        run: Copy-Item ./* C:\WebServer\CompanyAPI -Recurse -Force
        continue-on-error: true

      - name: Retry Deploy (second attempt)
        id: deploy-second-attempt
        if: steps.deploy-first-attempt.outcome == 'failure'
        run: |
          Start-Sleep -s 5
          Copy-Item ./* C:\WebServer\CompanyAPI -Recurse -Force
        continue-on-error: true

      - name: Retry Deploy (third attempt)
        if: steps.deploy-second-attempt.outcome == 'failure'
        run: |
          Start-Sleep -s 5
          Copy-Item ./* C:\WebServer\CompanyAPI -Recurse -Force

      # 3. Bring back online
      - name: Bring the app back online
        if: always()
        run: Remove-Item C:\WebServer\CompanyAPI\app_offline.htm
```

### Environment-Specific Deployment Strategies

**Dev/Test Environments** (develop & main branches): Full IIS management workflow
```powershell
# Full shutdown process
Stop-WebAppPool -Name "DevAPI"
iisreset /stop

# Triple retry deployment
Copy-Item ./* C:\WebServer\DevAPI -Recurse -Force

# Full startup process
Start-Service -Name WAS
Set-Service -Name WAS -StartupType Automatic
iisreset
Start-WebAppPool -Name "DevAPI"
```

**Production Environment** (release-triggered): Stability-first simplified workflow
```powershell
# Graceful shutdown
New-Item -Type File -Name app_offline.htm -Path C:\WebServer\API -Force

# Triple retry deployment mechanism
Copy-Item ./* C:\WebServer\API -Recurse -Force

# Auto recovery
Remove-Item C:\WebServer\API\app_offline.htm
```

### Technical Understanding & Practical Experience
- **Environment isolation:** Different deployment strategies per environment, balancing reliability and maintainability
- **Fault tolerance:** Multi-layered protection with triple retry and automatic rollback
- **Windows ecosystem integration:** Hands-on experience with .NET + IIS + Windows Server professional deployment
- **Deployment strategy optimization:** Production prioritizes stability; development prioritizes completeness

---

## Service Integration

### FlexMessage Service Encapsulation

**Service design based on actual FlexMessageService.cs:**

```csharp
using Newtonsoft.Json;
using System.Text;

namespace ExampleCompanyAPI.Services
{
    /// <summary>
    /// Flex Message Service
    /// Encapsulates outbound HTTP POST calls, responsible for pushing specified payloads
    /// as Flex Messages to a third-party notification API, with complete error handling
    /// and structured logging.
    /// </summary>
    public class FlexMessageService
    {
        private readonly HttpClient _httpClient;
        private readonly ILogger<FlexMessageService> _logger;

        public FlexMessageService(HttpClient httpClient, ILogger<FlexMessageService> logger)
        {
            _httpClient = httpClient;
            _logger = logger;
        }

        /// <summary>
        /// Send a Flex Message.
        /// </summary>
        /// <param name="apiUrl">Third-party notification API URL</param>
        /// <param name="payload">Object to push, serialized to JSON</param>
        /// <returns>
        /// (bool IsSuccess, string ErrorMessage)
        /// Returns success status and error message on failure (null if successful).
        /// </returns>
        public async Task<(bool IsSuccess, string ErrorMessage)> SendFlexMessageAsync(
            string apiUrl, object payload)
        {
            try
            {
                _logger.LogInformation("Sending Flex Message, API URL: {ApiUrl}", apiUrl);

                // Serialize and wrap as HTTP content
                var content = new StringContent(
                    JsonConvert.SerializeObject(payload),
                    Encoding.UTF8,
                    "application/json"
                );

                // Send POST request
                var response = await _httpClient.PostAsync(apiUrl, content);

                if (response.IsSuccessStatusCode)
                {
                    _logger.LogInformation("Flex Message sent successfully, API URL: {ApiUrl}", apiUrl);
                    return (true, null);
                }
                else
                {
                    var responseContent = await response.Content.ReadAsStringAsync();
                    _logger.LogError(
                        "Flex Message delivery failed, API URL: {ApiUrl}, Reason: {Reason}",
                        apiUrl,
                        responseContent
                    );
                    return (false, responseContent);
                }
            }
            catch (Exception ex)
            {
                _logger.LogError(
                    ex,
                    "Exception occurred while sending Flex Message, API URL: {ApiUrl}",
                    apiUrl
                );
                return (false, ex.Message);
            }
        }
    }
}
```

---

## Key Technical Implementations

### Business Workflow Control Architecture
**Challenge:** Managing the complete submission lifecycle across multi-plan differences, dual-layer review, and permission control.

**Technical Solutions:**
- **Flexible status design:** Each status includes CanEdit/CanReview/CanReject permission fields, supporting dynamic business rules
- **Strict transition validation:** Prevents illegal state changes, ensuring business process integrity
- **Complete audit mechanism:** Records every change with operator, timestamp, and reason

### Secure File Handling System
**Technical Implementation:**
- **Multi-layer validation strategy:** Hierarchical checks for file size (5MB), format, and quantity (3 images)
- **Secure filename handling:** GUID renaming prevents path traversal attacks; cross-platform path normalization
- **Resource management optimization:** `using` syntax ensures proper file stream disposal

### DevOps Deployment Architecture Participation
**Practical Results:**
- **Multi-environment strategy:** Differentiated deployment processes for dev/test/production
- **Triple retry mechanism:** Ensures deployment reliability with automatic rollback
- **Windows ecosystem integration:** Professional deployment with IIS + Application Pool + WAS services

---

## Professional Technical Competencies

### Core Technical Skills
- **System architecture design:** 13 API endpoints, 8 core entities in a complete backend architecture
- **Multi-system integration:** Dual LINE Bot + Web API + SQL Server ecosystem integration
- **Performance optimization:** EF Core Include() for eager loading, data projection, permission-scoped queries
- **Security practices:** JWT authentication, file upload security, SQL injection prevention

### Engineering Practices
- **Code quality:** Controllers maintain good readability and maintainability
- **Technical documentation:** Complete API documentation, architecture design documents, CI/CD workflow records
- **Problem solving:** Independently handling complex business logic, technical architecture, and deployment challenges
- **Cross-functional collaboration:** Close coordination with UI/UX designers and frontend engineers

### Business Value Created
- **System reliability:** 100% automated CI/CD, zero-downtime deployment, intelligent rollback mechanism
- **User experience optimization:** Seamless LINE Bot integration aligned with Taiwanese user habits
- **Operational efficiency:** Automated dual-layer review, personalized push notifications, comprehensive error handling

---

**Tech Stack:** .NET 6, Entity Framework Core 6, SQL Server 2022  
**Deployment Environment:** Windows Server 2022 + IIS 10  
**Codebase Scale:** 13 API endpoints + complete project architecture

---

> **Core Value Proposition:** This project demonstrates my core technical capabilities in backend development. The hands-on experience in **team CI/CD deployment processes** and **complex business logic handling** proves my ability to solve real-world business problems with technical expertise.

> **Technical Competency Demonstrated:** From Web API development to status management, security authentication, and CI/CD participation, this project showcases my core capabilities in **backend development** and **problem solving**.
