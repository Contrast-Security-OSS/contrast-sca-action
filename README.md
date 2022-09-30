# Use Contrast SCA to find your vulnerable dependencies
This GitHub action lets you use Contrast to detect vulnerable libraries in your code. It looks at project configuration files included in your code, identifies vulnerable dependencies and provides guidance on the versions to update.
- **Supported languages and their requirements:** 
  - **Java:** pom.xml and Maven build platform including the dependency plugin       
    *or* build.gradle and gradle dependencies or ./gradlew dependencies must be     
    supported                                                                     
  - **.NET core:** MSBuild 15.0 or greater and a                   
    packages.lock.json file.                                                      
    Note: If the packages.lock.json file is unavailable it can be generated by    
    setting RestorePackagesWithLockFile to true within each *.csproj file and     
    running dotnet build.
  - **Node:** package.json and a lock file (either .package-lock.json or .yarn.lock.)
  - **Ruby:** gemfile and gemfile.lock
  - **Python:** pipfile and pipfile.lock
  - **Go:** go.mod
  - **PHP:** composer.json and composer.lock
- **CodeSec by Contrast users:** Retrieve authentication details using the command line tool - CodeSec by Contrast.
  - Installation instructions here : [https://www.contrastsecurity.com/developer/codesec](https://www.contrastsecurity.com/developer/codesec)
  - Use the 'contrast auth' and 'contrast config' commands to collect the required credentials.
- **Licensed Contrast users:** Get these credentials from the user area Contrast web interface:
  - Authorization header
  - API key
  - Organization ID
## Required inputs
- apiKey - An API key from the Contrast platform.
- authHeader - User authorization credentials from Contrast.
- orgId - The ID of your organization in Contrast.
- filePath - Specify the directory in which to search for project configuration files.
## Optional inputs
- apiUrl - The URL of the host. This input includes the protocol section of the URL (https://). The default value is `https://ce.contrastsecurity.com` (Contrast Community Edition).
- severity - Allows user to report libraries with vulnerabilities above a chosen severity level. Values for level are high, medium or low. (Note: Use this input in combination with the fail input, otherwise the action will exit)
- fail - When set to true, fails the action if CVEs have been detected that match at least the severity option specified.
- ignoreDev - When set to true, excludes developer dependencies from the results.
## Usage
All Contrast-related account secrets should be configured as GitHub secrets and will be passed via environment variables in the GitHub runner.
A simple workflow to get going is:
```yaml
on:
  push:
    branches:
      - "main"
jobs:
  perform-sca-audit:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Set up JDK 11
        uses: actions/setup-java@v3
        with:
          java-version: '11'
          distribution: 'adopt'

      - name: build jar
        run: |
          mvn clean install -DskipTests

      - name: Contrast SCA Audit Action
        uses: Contrast-Security-OSS/contrast-sca-action@v1
        with:
          apiKey: ${{ secrets.CONTRAST_API_KEY }}
          orgId: ${{ secrets.CONTRAST_ORGANIZATION_ID }}
          authHeader: ${{ secrets.CONTRAST_AUTH_HEADER }}
          apiUrl: ${{ secrets.CONTRAST_API_URL }}
          filePath: mypath/to/config/files
          severity: medium
          fail: true
```

## Initial steps for using the action
These instructions assume you already have set up a GitHub workflow to build your project.  If not, read the
[GitHub Actions](https://docs.github.com/en/actions) documentation to learn what GitHub Actions are and how to set them
up. After understanding what a GitHub action is, then come back here to complete the following steps:
1. Create a branch of your code to add the Contrast Audit action to your workflow. This branch is typically located at `./github/workflows/build.yml`
2. Add the `contrastaudit-action` to your workflow and commit.
3. After committing, create a Pull Request (PR) to merge the update back to your main branch. Creating the PR triggers the Contrast SCA action to run. The extra "Code Scanning" check appears in the PR.
