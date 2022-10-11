# Use Contrast SCA to find your vulnerable dependencies
This GitHub action lets you use Contrast to detect vulnerable libraries in your code. The action looks at project configuration files included in your code, identifies vulnerable dependencies and provides guidance on the versions to update.

## Initial steps for using the action
If you are not familiar with GitHub actions read the
[GitHub Actions](https://docs.github.com/en/actions) documentation to learn what GitHub Actions are and how to set them
up. After which, complete the following steps:

1. Configure the following GitHub secrets CONTRAST_API_KEY, CONTRAST_ORGANIZATION_ID, CONTRAST_AUTH_HEADER and CONTRAST_API_URL 

- **CodeSec by Contrast users:** Retrieve authentication details using the CLI.
  - Installation instructions here : [https://www.contrastsecurity.com/developer/codesec](https://www.contrastsecurity.com/developer/codesec)
  - If you are a new user, create an account with the 'contrast auth' command
  - The 'contrast config' command can then be used to collect the required credentials
  
    <img width="593" alt="image" src="https://user-images.githubusercontent.com/24421341/195069309-0cd48676-f7a8-465a-aa8c-f5fc117d7c88.png">
    
- **Licensed Contrast users:** Get your credentials from the 'User Settings' menu in the Contrast web interface: You will need the following 
  - Organization ID
  - Your API key
  - Authorization header
  - You will also need the URL of your Contrast UI host. This input includes the protocol section of the URL (https://). The default value is `https://ce.contrastsecurity.com` (Contrast Community Edition).
  
    <img width="660" alt="image" src="https://user-images.githubusercontent.com/24421341/195047954-6ade693f-a911-4914-b50c-5121fd77dcae.png">

2. Copy one of the sample workflows below and create a branch of your code to add the Contrast SCA action to your workflow. This branch is typically located at `.github/workflows/build.yml`
3. Update the workflow file to specify when the action should run (for example on pull_request, on push)
   
    ```yaml
    on:
      pull_request:
        branches:
          - "main"
    ```
5. Update the filepath in the workflow file to specfy the location of the project configuration file where dependencies are declared

    ```yaml
              filePath: package.json
    ```

7. To fail based on severity of CVEs found set severity  (critical/high/medium or low) and fail to true

    ```yaml
              severity: medium
              fail: true
    ```
    
9. After committing, create a Pull Request (PR) to merge the update back to your main branch. Creating the PR triggers the Contrast SCA action to run. The extra "Code Scanning" check appears in the PR.



## Usage

The following are sample workflows to get started in Java, Node, PHP.

### Java

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

### Node

```yaml
name: SCA Node

on:
  pull_request:
    branches:
      - "main"

jobs:
  perform-sca-node:
    runs-on: ubuntu-latest
    steps:
        # Checkout/build your application/install Node
      - uses: actions/checkout@v3

      - name: Contrast SCA Action
        uses: Contrast-Security-OSS/contrast-sca-action@main
        with:
          apiKey: ${{ secrets.CONTRAST_API_KEY }}
          orgId: ${{ secrets.CONTRAST_ORGANIZATION_ID }}
          authHeader: ${{ secrets.CONTRAST_AUTH_HEADER }}
          apiUrl: ${{ secrets.CONTRAST_API_URL }}
          filePath: mypath/to/config/files
          severity: medium
          fail: true

```

### PHP

```yaml
name: SCA PHP

on:
  push:
    branches:
      - "main"

jobs:
  perform-sca-php:
    runs-on: ubuntu-latest
    steps:
        # Check out/build your application
      - uses: actions/checkout@v3

        # Install composer
      - uses: php-actions/composer@v6

      - name: Contrast SCA Action
        uses: Contrast-Security-OSS/contrast-sca-action@main
        with:
          apiKey: ${{ secrets.CONTRAST_API_KEY }}
          orgId: ${{ secrets.CONTRAST_ORGANIZATION_ID }}
          authHeader: ${{ secrets.CONTRAST_AUTH_HEADER }}
          apiUrl: ${{ secrets.CONTRAST_API_URL }}
          filePath: mypath/to/config/files
          severity: medium
          fail: true

```

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

All Contrast-related account secrets should be configured as GitHub secrets and will be passed via environment variables in the GitHub runner.

## Required inputs
- apiKey - An API key from the Contrast platform.
- authHeader - User authorization credentials from Contrast.
- orgId - The ID of your organization in Contrast.
- filePath - Specify the path for project configuration file (e.g. lib/package.json) .
- apiUrl - Required for Licensed Contrast Users only. This input includes the protocol section of the URL (https://). The default value is `https://ce.contrastsecurity.com` (Contrast Community Edition).
## Optional inputs
- severity - Allows user to report libraries with vulnerabilities above a chosen severity level. Values for level are high, medium or low. (Note: Use this input in combination with the fail input, otherwise the action will exit)
- fail - When set to true, fails the action if CVEs have been detected that match at least the severity option specified.
- ignoreDev - When set to true, excludes developer dependencies from the results.
