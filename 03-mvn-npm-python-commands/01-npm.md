# 01. npm (Node Package Manager)
      - What it is: The default package manager for Node.js applications that installs, updates, and manages dependencies (libraries) in JavaScript 
        projects.


# 02. Commands
      
      a. npm init
         - Creates a package.json file by asking questions about your project
         - Analogy: Writing a Shopping list
      
      b. npm install
         - Installs all dependencies listed in package.json
         - Analogy: Buying every item on the list
      
      c. npm install <pkg>
         - Installs a specific package locally and adds it to package.json
         - Analogy: Buying one item and noting it
      
      d. npm install -g <pkg>
         - Installs a package globally for commands you can run anywhere
         - Analogy: Buying a tool and storing it centrally
      
      e. npm uninstall <pkg>
         - Removes a package from the project and updates package.json
         - Analogy: Returning an item you no longer need
      
      f. npm update
         - Updates all outdated packages in your project	
         - Analogy: Checking for newer versions of items
      
      g. npm run <script>
         - Runs a script defined in the scripts section of package.json, e.g. "test": "mocha"	
         - Analogy: Following a custom recipe step
      
      h. npm ls
         - Lists installed packages and their versions	
         - Analogy: Inventorying your pantry
      
      i. npm outdated
         - Shows which installed packages have newer versions available	
         - Analogy: Checking expiration dates


# 03. Development Environment Commands
      
      a. npm install
         - Install or update dependencies from package.json
         - updates package-lock.json like shopping for ingredients.
      
      b. npm install <pkg> --save[-dev]
         - Add a new library or dev-tool to your project (e.g., npm install eslint --save-dev).
      
      c. npm update
         - Update outdated packages to latest semver-compatible versions.
      
      d. npm outdated
         - List packages with newer versions available.
      
      e. npm run <script>
         - Run custom scripts defined in your package.json (tests, linting, builds).
      
      f. npm audit
         - Scan for known vulnerabilities in dependencies .
      
      g. npm audit fix
         - Automatically apply nonbreaking security fixes.


# 04. CI & Testing Environment Commands
      
      a. npm ci --no-audit --no-fund --loglevel=error
         - Fast, reproducible “clean install” using package-lock.json; removes node_modules and silences noise 
      
      b. npm test
         - Execute test suite (unit/integration) as defined in scripts.
      
      c. npm run lint
         - Run linters or code analyzers to enforce style and catch issues early.
      
      d. npm audit --audit-level=high
         - Fail builds on vulnerabilities of a specified severity.
      
      e. npm prune --production
         - Remove devDependencies before packaging artifacts.


# 05. Staging Environment Commands
      
      a. npm ci
         - Reinstall exactly locked dependencies in an environment mirroring production.
      
      b. npm run build
         - Compile or bundle source code (e.g., webpack, TypeScript).
      
      c. npm prune --production
         - Strip out devDependencies to reduce image size.
      
      d. npm audit
         - Final security check before deployment.


# 06. Production Environment Commands
      
      a. NODE_ENV=production npm ci or npm install --production
         - Install only production dependencies; skip devDependencies for minimal footprint.
      
      b. npm prune --production
         - Extra safeguard to remove any leftover devDependencies.
      
      c. npm audit --production --audit-level=moderate	
         - Periodic vulnerability scanning in production.
      
      d. npm run start
         - Launch the application via its start script (e.g., entrypoint).



