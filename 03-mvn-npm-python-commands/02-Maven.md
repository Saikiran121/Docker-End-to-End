# 01. Development Environment Commands
      
      a. mvn clean compile
         - Remove previous builds and compile source code
         - like clearing your workbench and preparing materials.
      
      b. mvn test
         - Run unit tests to validate code changes during development.
      
      c. mvn compile test-compile
         - Compile both main and test sources without running tests.
      
      d. mvn dependency:tree
         - Visualize dependency hierarchy to understand transitive dependencies and resolve conflicts.
      
      e. mvn dependency:analyze
         - Identify unused dependencies and missing direct dependencies.
      
      f. mvn versions:display-dependency-updates
         - Show available updates for project dependencies.
      
      g. mvn spring-boot:run or mvn exec:java
         - Run application locally for development testing.


# 02. CI & Testing Environment Commands
      
      a. mvn clean test -B -ntp
         - Clean install with batch mode (-B) and no transfer progress (-ntp) for CI optimization.
      
      b. mvn clean verify -B -ntp
         - Run full test suite including integration tests and verify packaging.
      
      c. mvn test -Dtest=*IntegrationTest
         - Run only integration tests or specific test patterns.
      
      d. mvn surefire-report:report
         - Generate test reports for CI dashboard integration.
      
      e. mvn dependency:check-for-updates	
         - Fail build if outdated dependencies pose security risks.
      
      f. mvn org.owasp:dependency-check-maven:check	
         - Scan dependencies for known security vulnerabilities.
      
      g. mvn clean package -DskipTests=false	
         - Package application while ensuring all tests pass.
      

# 03. Staging Environment Commands
      
      a. mvn clean package -Pstaging -DskipTests=true	
         - Build with staging profile, skipping tests (already validated in CI).
      
      b. mvn clean install -Pstaging	
         - Install staging artifacts to local repository for deployment preparation.
      
      c. mvn dependency:copy-dependencies	
         - Copy all dependencies to target folder for containerization.
      
      d. mvn spring-boot:build-image -Pstaging	
         - Build optimized container image with staging configuration.
      
      e. mvn versions:set -DnewVersion=1.0.0-SNAPSHOT	
         - Set version for staging releases.


# 04. Production Environment Commands
      
      a. mvn clean package -Pprod -DskipTests=true -q	
         - Build production artifacts quietly, skipping tests (validated in previous stages).
      
      b. mvn install:install-file (for artifacts)	
         - Install production artifacts to deployment repositories.
      
      c. mvn dependency:purge-local-repository	
         - Clean local repository of snapshots before production deployment.
      
      d. mvn versions:set -DnewVersion=1.0.0	
         - Set final release version (remove SNAPSHOT).
      
      e. mvn clean deploy -Pprod -DskipTests=true	
         - Deploy to production artifact repository (Nexus, Artifactory).

