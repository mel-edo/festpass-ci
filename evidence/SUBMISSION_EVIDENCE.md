# FestPass Jenkins CI/CD Challenge: Submission Evidence & Summary

## 1. Student & Lab Metadata

| Field | Value |
|---|---|
| **Student Alias** | `mel-edo` *(read from Git commit author)* |
| **Class Code** | `001` |
| **Challenge Code** | `00264` |
| **Installation ID** | `0e6ac8ac-e605-45af-a8ee-e5e3fd5e67f4` |
| **Machine Label Hash** | `f55d3cc0022a3b09d2e08f55e565a6a55fe829c79c95f716ed784e18863c40cb` |
| **Git Repository** | `git://127.0.0.1:9418/festpass-java` |
| **Branch** | `main` |
| **Jenkins Job Name** | `festpass-ci` |
| **Dedicated Build Agent** | `student-laptop` (1 executor; built-in node = 0 executors) |

---

## 2. Build Lifecycle & SCM Polling Trace

### Summary Table

| Build # | Git Commit | Build Trigger / Cause | Test Results | Pipeline Status | Published Artifacts |
|:---:|:---:|:---:|:---:|:---:|:---:|
| **Build 1** | `59f6854` | Manual (`Build Now`) | 25 pass, 3 fail | **FAILURE (RED)** | `submission-evidence.csv` |
| **Build 2** | `db5733c` | **Started by an SCM change** | 25 pass, 3 fail | **FAILURE (RED)** | `submission-evidence-red.csv` |
| **Build 3** | `8cde3a3` | **Started by an SCM change** | 28 pass, 0 fail | **SUCCESS (GREEN)** | `festpass.jar`, `smoke-test.txt`, `build-info.txt`, `submission-evidence-green.csv` |

---

## 3. Red Build Diagnosis (Build #2)

* **Commit ID**: `db5733c6d34330115f3536edc34ccc4661829605`
* **Commit Message**: `Personalize my help introduction`
* **Trigger Cause**: `Started by an SCM change` (Automatic Jenkins `pollSCM('* * * * *')` polling)
* **Skipped Product Stages**: `Build JAR`, `Smoke test JAR`, `Save artifact`

### Failing Assertions (3 Failures)
1. **`FestPassTest.cancellationFreesSeat`**:
   - *Error*: `java.lang.IllegalArgumentException: SOLD OUT. Even the developer needs a ticket.`
2. **`FestPassTest.cancellationsSurviveReload`**:
   - *Error*: `java.lang.AssertionError: Cancellation was lost`
3. **`FestPassTest.cliWorkflowPersistsAcrossCommands`**:
   - *Error*: `java.lang.AssertionError: List does not reflect persisted cancellation`

### Violated Business Rule
When a student cancels an existing event registration via `cancel(eventId, studentId)`, the application must remove the student from the event's attendee roster. Removing the attendee decrements the booked count and restores available seat capacity. 

In the starter code for `BookingService.java`, the `cancel` method verified that the student held an existing booking but never invoked `remove` on the `students` set. Because the student remained on the roster, the seat was never released. When another student attempted to book the supposedly freed seat on an event at full capacity, the booking was rejected with `SOLD OUT`.

---

## 4. Application Repair (Build #3)

* **Commit ID**: `8cde3a3a5ca100cfde27b41f557497eb234e20d6`
* **Commit Message**: `Fix cancellation to remove student from event roster and restore seat availability`
* **Trigger Cause**: `Started by an SCM change` (Automatic Jenkins polling)
* **Status**: **SUCCESS (GREEN)** &mdash; All 28 tests passed (0 failures, 0 errors, 0 skipped).
* **Completed Stages**: `Checkout` &rarr; `Submission setup` &rarr; `Test` &rarr; `Build JAR` &rarr; `Smoke test JAR` &rarr; `Save artifact`

### Minimal Code Fix
File: [`src/main/java/BookingService.java`](file:///home/meledo/college_files/sem7/cloud/expert/Share_With_Students/CloudJenkinsLab/festpass-java/src/main/java/BookingService.java)
```diff
--- a/src/main/java/BookingService.java
+++ b/src/main/java/BookingService.java
@@ -65,7 +65,7 @@ public final class BookingService {
     public void cancel(String eventId, String studentId) {
         validateId(studentId);
         Registration event = requireEvent(eventId);
-        if (!event.students.contains(studentId)) {
+        if (!event.students.remove(studentId)) {
             throw new IllegalArgumentException("No booking for student: " + studentId);
         }
     }
```

---

## 5. Final Executable Artifact Verification

### Executable JAR Details
* **Artifact Path**: `build/festpass.jar`
* **SHA-256 Checksum**: `ef703030cf93c36b873b3fcd10ebec04b9068bc73377db5c1ea8bce383b6ec3d`

### Executable Command Output (`java -jar festpass.jar help`)
```text
FestPass - campus event registration by mel-edo
Usage: java -jar build/festpass.jar [--data FILE] COMMAND
  add-event ID "TITLE" CAPACITY
  list
  book EVENT_ID STUDENT_ID
  cancel EVENT_ID STUDENT_ID
  roster EVENT_ID
  summary
  help
Default data file: data/festpass.tsv
IDs are case-sensitive. Use synthetic student IDs, not personal data.
Exit codes: 0 success, 2 invalid request, 3 storage failure.
```

### Full Smoke Test Walkthrough
```text
EVENT CREATED: java
BOOKING ACCEPTED: S1 -> java
Invalid input: SOLD OUT. Even the developer needs a ticket.
BOOKING CANCELLED: S1 -> java
BOOKING ACCEPTED: S2 -> java
Events: 1 | Booked: 1 | Available: 0
```
All seven smoke verification checks passed.
