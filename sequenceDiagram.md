# Sequence Diagram - Main Flow: Job Application with Status Update

```mermaid
sequenceDiagram
    participant JS as Job Seeker
    participant FE as Frontend
    participant API as Backend API
    participant DB as Database
    participant ES as Email Service
    participant Rec as Recruiter

    %% Phase 1: Authentication
    rect rgb(200, 220, 240)
        Note over JS,API: Phase 1: User Registration & Login
        JS->>FE: 1. Enter credentials (register)
        FE->>API: 2. POST /api/auth/register
        API->>DB: 3. Create user record
        DB-->>API: 4. User created
        API-->>FE: 5. Return JWT token
        FE-->>JS: 6. Login successful
    end

    %% Phase 2: Browse Jobs
    rect rgb(220, 240, 200)
        Note over JS,API: Phase 2: Browse & Search Jobs
        JS->>FE: 7. Browse job listings
        FE->>API: 8. GET /api/jobs?filter=...
        API->>DB: 9. Query job listings
        DB-->>API: 10. Return jobs
        API-->>FE: 11. Return filtered jobs
        FE-->>JS: 12. Display jobs list
    end

    %% Phase 3: View Job Details
    rect rgb(240, 240, 200)
        Note over JS,API: Phase 3: View Job Details
        JS->>FE: 13. Click on job
        FE->>API: 14. GET /api/jobs/:jobId
        API->>DB: 15. Fetch job details
        DB-->>API: 16. Return full job info
        API-->>FE: 17. Return job details
        FE-->>JS: 18. Display job description
    end

    %% Phase 4: Apply for Job
    rect rgb(240, 220, 220)
        Note over JS,API: Phase 4: Submit Application
        JS->>FE: 19. Click "Apply" & upload resume
        FE->>API: 20. POST /api/applications (with resume)
        API->>DB: 21. Validate user & job exist
        DB-->>API: 22. Validation passed
        API->>DB: 23. Create application record
        DB-->>API: 24. Application ID returned
        API->>DB: 25. Create ApplicationStatus (Applied)
        DB-->>API: 26. Status logged
        API->>ES: 27. Trigger email (Application Received)
        ES-->>API: 28. Email queued
        API-->>FE: 29. Return confirmation
        FE-->>JS: 30. Show success message
    end

    %% Phase 5: Email Sent
    rect rgb(255, 230, 200)
        Note over API,ES: Phase 5: Email Notification to Job Seeker
        ES->>JS: 31. Send "Application Received" email
        JS-->>ES: 32. Email delivered
    end

    %% Phase 6: Recruiter Reviews
    rect rgb(230, 220, 240)
        Note over Rec,API: Phase 6: Recruiter Reviews Applications
        Rec->>FE: 33. Login & view applications
        FE->>API: 34. GET /api/jobs/:jobId/applications
        API->>DB: 35. Fetch all applications
        DB-->>API: 36. Return applications list
        API-->>FE: 37. Return applications
        FE-->>Rec: 38. Display applications
    end

    %% Phase 7: Update Status
    rect rgb(240, 230, 220)
        Note over Rec,API: Phase 7: Update Application Status
        Rec->>FE: 39. Click "Shortlist" on application
        FE->>API: 40. PUT /api/applications/:appId/status
        API->>DB: 41. Validate current status
        DB-->>API: 42. Status validation passed
        API->>DB: 43. Update application status
        DB-->>API: 44. Status updated
        API->>DB: 45. Log ApplicationStatus change
        DB-->>API: 46. Status audit logged
        API->>ES: 47. Trigger email (Status Changed)
        ES-->>API: 48. Email queued
        API-->>FE: 49. Return updated data
        FE-->>Rec: 50. Show confirmation
    end

    %% Phase 8: Job Seeker Notified
    rect rgb(200, 240, 220)
        Note over API,ES: Phase 8: Email Notification to Job Seeker
        ES->>JS: 51. Send "Shortlisted" email
        JS-->>ES: 52. Email delivered
    end

    %% Phase 9: Check Status
    rect rgb(240, 240, 240)
        Note over JS,API: Phase 9: Check Application Status
        JS->>FE: 53. Check application status
        FE->>API: 54. GET /api/applications/:appId
        API->>DB: 55. Fetch application & status
        DB-->>API: 56. Return application data
        API-->>FE: 57. Return application details
        FE-->>JS: 58. Display updated status
    end
```

## End-to-End Flow Summary

### Main Workflow Phases

1. **Authentication (Steps 1-6)**
   - Job seeker registers/logs in
   - Backend validates credentials and returns JWT token
   - Frontend stores token for authenticated requests

2. **Browse & Search (Steps 7-12)**
   - User browses available job listings
   - Can filter by title, location, experience level, etc.
   - Backend returns filtered results from database

3. **View Details (Steps 13-18)**
   - User clicks on specific job
   - Backend fetches full job details
   - User sees complete job description and requirements

4. **Apply for Job (Steps 19-30)**
   - User clicks apply and uploads resume
   - Frontend sends application with resume file
   - Backend validates user and job exist
   - Backend creates application record
   - Backend logs initial status as "Applied"
   - Email service is triggered

5. **Email Confirmation (Steps 31-32)**
   - Email system sends confirmation to job seeker
   - Job seeker receives application receipt email

6. **Recruiter Reviews (Steps 33-38)**
   - Recruiter logs in and views applications for their jobs
   - Backend fetches all applications for that job
   - Recruiter sees list of candidates

7. **Update Status (Steps 39-50)**
   - Recruiter changes application status (e.g., to "Shortlisted")
   - Backend validates status transition is allowed
   - Backend updates application record
   - Backend logs status change in audit trail
   - Email service is triggered with new status

8. **Status Notification (Steps 51-52)**
   - Email system sends status update to job seeker
   - Job seeker is notified of progress

9. **Check Status (Steps 53-58)**
   - Job seeker can check application status anytime
   - Backend returns current application state with history

## Key Design Decisions

- **Async Email**: Emails are queued asynchronously to avoid blocking the API
- **Status Audit Trail**: Every status change is logged for transparency
- **Validation**: Status transitions are validated (can't go backward)
- **JWT Auth**: Token-based authentication for API security
- **Separation of Concerns**: Email logic separated into service
- **Database Transactions**: Application creation is atomic
