FestPass Docker & Container Security Assessment Report
Registration number: 202401100200340
Application: FestPass 2026 Registration Portal
Backend image under initial scan: fest-backend:0.1
Frontend: HTML/CSS/JavaScript served by Nginx
Scan tool: Trivy
Report status: Draft based on the supplied scan screenshot; complete counts and final gate status must be copied from the actual terminal output.
1. Objective
The objective is to containerize a small registration application, connect the frontend, API, and PostgreSQL database using Docker Compose, and assess the backend image for known vulnerabilities. The frontend communicates with the API through Nginx's /api/ reverse proxy, so the browser does not need direct access to the database.
2. Frontend implementation
The frontend in frontend/index.html provides:
- A responsive registration form with name and event-category fields.
- A backend health indicator that calls GET /api/health.
- A recent-registration list that calls GET /api/registrations.
- A submission action that sends JSON to POST /api/registrations.
- Visible success/error messages and a refresh control.
frontend/nginx.conf serves the static page and forwards /api/ requests to the Compose service named backend on port 5000.

API compatibility check: confirm that the backend's POST endpoint accepts {"name":"…","event":"…"} and that GET /api/registrations returns either a JSON array or an object with a registrations/items array. Adjust the field names if the supplied backend uses a different schema.
3. Initial Trivy scan
The supplied screenshot shows Trivy reporting vulnerabilities in Python-related packages and metadata, including Werkzeug, jaraco.context, pip, setuptools, and wheel. The visible excerpt includes findings marked HIGH, MEDIUM, and LOW. Examples visible in the screenshot include:
- jaraco.context — a HIGH-severity path-traversal finding.
- wheel — a HIGH-severity finding.
- pip and setuptools — multiple findings with differing severities.
- Werkzeug — several Windows-path-related findings and a LOW-severity cookie-related finding.
The screenshot is a partial table, so this report does not claim a total vulnerability count. Copy the actual totals from the full Trivy output or the saved reports/trivy-before.txt file.
4. Remediation plan
1. Replace the broad, floating base image with a maintained, explicitly pinned Python slim image appropriate to the environment.
2. Pin compatible, patched application dependencies in requirements.txt; upgrade installer/build tooling only as required and test the application after changes.
3. Run the application as a non-root user.
4. Keep secrets out of the image and source control. Supply database credentials via the local .env file and ensure it is ignored by Git.
5. Rebuild the hardened image as fest-backend:0.2 and scan it again with Trivy.
6. Enforce the requested gate against the actual image:trivy image --severity CRITICAL --ignore-unfixed --exit-code 1 fest-backend:0.2
   A successful gate should return exit code 0; verify the real exit code rather than assuming it passed.
5. Before/after comparison
Check	Initial image fest-backend:0.1	Hardened image fest-backend:0.2
Trivy scan	Run completed; screenshot shows findings	Run and save output to reports/trivy-after.txt
Critical count	Fill from full scan output	Fill from full scan output
High count	Fill from full scan output	Fill from full scan output
Medium count	Fill from full scan output	Fill from full scan output
Low count	Fill from full scan output	Fill from full scan output
CRITICAL gate	Not established by supplied excerpt	Record actual command exit code


6. Validation checklist
- [ ] Frontend loads from host port 8040.
- [ ] /api/health reports the expected registration number and database status.
- [ ] A registration can be submitted and retrieved from PostgreSQL.
- [ ] Data remains after docker compose down followed by docker compose up -d (do not use -v for this persistence check).
- [ ] Frontend cannot directly access the database service/network.
- [ ] reports/trivy-before.txt and reports/trivy-after.txt contain genuine command output.
- [ ] Docker image tags and registry digest are copied from actual Docker output.
- [ ] Signing and verification results are documented only after successful real commands.
- [ ] .env and private signing keys are excluded from Git; only the public signing key may be committed.
7. Evidence and integrity note
Use genuine screenshots from your own terminal for assessment evidence. The screenshots supplied for reference show a scan excerpt but do not establish the final scan totals, hardened-image results, registry digest, or signing status. Do not enter guessed counts or claim a security gate passed unless the command output confirms it.