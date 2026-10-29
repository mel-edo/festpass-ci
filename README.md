# festpass

Campus event registration service with Jenkins automated CI/CD pipeline.

## Build and Run

```bash
# Verify tests and build
java ci/Build.java verify

# Run smoke tests
java ci/SmokeTest.java

# Run the application
java -jar build/festpass.jar help
```

## Evidence

Submission reports, red/green evidence CSVs, and artifacts are stored in [`evidence/`](evidence/).
