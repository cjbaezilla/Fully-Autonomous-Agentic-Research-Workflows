# Top 5 NPM Packages for Building CLIs

My top recommendations for building command-line interfaces in Node.js:

| Package | Best For | Pros | Cons |
|---------|----------|------|------|
| **Commander.js** | Command structure & parsing | Intuitive API, excellent subcommand support, great docs, battle-tested | Less flexible than yargs for complex scenarios |
| **Inquirer** | Interactive prompts | Simple async/await API, wide variety of prompt types, highly customizable | Bundle size can be large; some users prefer Enquirer as lighter alternative |
| **Chalk** | Terminal colors/styling | Zero-dependency, chainable API, works with Windows consoles | Styles limited to basic colors/formatting |
| **Ora** | Spinners & progress | Beautiful spinners, auto-detects TTY, customizable | Spinner-only; no progress bars (use cli-progress separately) |
| **Yargs** | Advanced parsing | Extremely flexible, powerful validation, command composition | Steeper learning curve, more verbose for simple CLIs |

## Honorable Mentions

- **cli-progress** - Dedicated progress bars (complements Ora)
- **figlet** - ASCII art banners
- **meow** - Minimal CLI helper for simple tools
- **enquirer** - Modern, lightweight alternative to Inquirer
- **update-notifier** - Add update notifications to your CLI

---

# Complete CLI Development Tutorial

This guide will take you from a simple CLI to production-ready tools.

## Prerequisites: Project Setup

### 1. Create a new project

```bash
mkdir my-cli && cd my-cli
npm init -y
```

### 2. Install core dependencies

```bash
npm install commander inquirer chalk ora cli-progress
npm install -D @types/inquirer typescript ts-node
```

### 3. Configure package.json

```json
{
  "name": "my-awesome-cli",
  "version": "1.0.0",
  "description": "A learning CLI project",
  "main": "dist/index.js",
  "bin": {
    "mycli": "./dist/index.js"
  },
  "scripts": {
    "build": "tsc",
    "dev": "ts-node src/index.ts",
    "start": "node dist/index.js",
    "lint": "eslint src/",
    "test": "jest"
  },
  "keywords": ["cli", "learning"],
  "author": "Your Name",
  "license": "MIT"
}
```

---

## Level 1: Basics - Your First CLI

### Simple Hello World

```javascript
#!/usr/bin/env node

console.log('Hello, World!');
```

Make executable and run:
```bash
chmod +x index.js
node index.js
```

### Basic Argument Parsing with Commander

```javascript
#!/usr/bin/env node

const { Command } = require('commander');

const program = new Command();

// Basic command with arguments
program
  .argument('<name>', 'name of person to greet')
  .option('-g, --greeting <type>', 'greeting style', 'hello')
  .action((name, options) => {
    console.log(`${options.greeting}, ${name}!`);
  });

program.parse();
```

**Run it:**
```bash
node index.js World
node index.js Alice --greeting hola
```

### Positional vs Optional Arguments

```javascript
program
  .argument('<firstName>')
  .argument('[middleName]')  // optional
  .argument('[lastName]')   // optional
  .action((firstName, middleName, lastName) => {
    const fullName = [firstName, middleName, lastName]
      .filter(Boolean)
      .join(' ');
    console.log(`Hello, ${fullName}!`);
  });
```

---

## Level 2: Commands & Subcommands

### Basic Commands

```javascript
const { Command } = require('commander');

const program = new Command();

program
  .name('git-tutorial')
  .description('Git-like CLI tutorial')
  .version('1.0.0');

// Add command
program.command('add <file>')
  .description('Add file to staging')
  .action((file) => {
    console.log(`Adding ${file} to staging area`);
  });

// Add with options
program.command('commit')
  .description('Commit changes')
  .option('-m, --message <msg>', 'commit message')
  .option('--amend', 'amend previous commit')
  .action((options) => {
    if (options.amend) {
      console.log('Amending previous commit...');
    } else {
      console.log(`Committing with message: ${options.message}`);
    }
  });

// Command with multiple arguments
program.command('branch <operation> [name]')
  .description('Create, list, or delete branches')
  .action((operation, name) => {
    switch(operation) {
      case 'create':
        console.log(`Created branch: ${name}`);
        break;
      case 'list':
        console.log('* main');
        console.log('  feature/example');
        break;
      case 'delete':
        console.log(`Deleted branch: ${name}`);
        break;
      default:
        console.log('Unknown operation');
    }
  });

program.parse();
```

### Nested Subcommands (Action Composition)

```javascript
const { Command } = require('commander');

const program = new Command();

// Parent command with shared logic
const createCommand = program.command('create <type>');

createCommand
  .argument('[name]', 'name of item')
  .option('--force', 'force creation')
  .action(async (type, name, options) => {
    console.log(`Creating ${type}: ${name || 'unnamed'}`);
    console.log(`Force: ${options.force}`);
    
    // Type-specific logic
    switch(type) {
      case 'project':
        // create project logic
        break;
      case 'branch':
        // create branch logic
        break;
    }
  });

program.parse();
```

---

## Level 3: Interactive Prompts with Inquirer

### Basic Prompts

```javascript
const inquirer = require('inquirer');

async function basicPrompts() {
  const answers = await inquirer.prompt([
    {
      type: 'input',
      name: 'username',
      message: 'What is your username?',
      default: 'guest'
    },
    {
      type: 'password',
      name: 'password',
      message: 'Enter your password:'
    }
  ]);
  
  console.log(answers);
}
```

### Validation

```javascript
const answers = await inquirer.prompt([
  {
    type: 'input',
    name: 'email',
    message: 'Enter email:',
    validate: function(input) {
      if (!/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(input)) {
        return 'Invalid email format';
      }
      return true;
    }
  },
  {
    type: 'number',
    name: 'age',
    message: 'Enter age:',
    validate: function(input) {
      if (input < 18) return 'Must be 18 or older';
      if (input > 120) return 'Invalid age';
      return true;
    }
  }
]);
```

### Confirm Prompt

```javascript
const { confirm } = await inquirer.prompt([
  {
    type: 'confirm',
    name: 'proceed',
    message: 'Delete all files?',
    default: false
  }
]);

if (confirm) {
  console.log('Proceeding with deletion...');
}
```

### List (Select) Prompt

```javascript
const { framework } = await inquirer.prompt([
  {
    type: 'list',
    name: 'framework',
    message: 'Select a framework:',
    choices: ['React', 'Vue', 'Angular', 'Svelte'],
    default: 'React'
  }
]);
```

### Checkbox (Multiple Selection)

```javascript
const { features } = await inquirer.prompt([
  {
    type: 'checkbox',
    name: 'features',
    message: 'Select features to install:',
    choices: [
      { name: 'ESLint', value: 'eslint', checked: true },
      { name: 'Prettier', value: 'prettier' },
      { name: 'TypeScript', value: 'typescript' },
      { name: 'Jest', value: 'jest' }
    ]
  }
]);

console.log(`Selected: ${features.join(', ')}`);
```

### Radio with Pagination (Large Lists)

```javascript
const { country } = await inquirer.prompt([
  {
    type: 'rawlist',
    name: 'country',
    message: 'Select country:',
    choices: [
      'United States',
      'Canada',
      'United Kingdom',
      'Germany',
      'France',
      // ... many more
    ]
  }
]);
```

### Editor Prompt

```javascript
const { details } = await inquirer.prompt([
  {
    type: 'editor',
    name: 'details',
    message: 'Edit your commit message:',
    default: 'Initial commit\n\nAdd more details here...'
  }
]);
```

### Dynamic Choices

```javascript
const { repo } = await inquirer.prompt([
  {
    type: 'list',
    name: 'repo',
    message: 'Select repository:',
    choices: async () => {
      const repos = await fetchRepos(); // your async function
      return repos.map(r => ({ name: r.name, value: r.url }));
    }
  }
]);
```

### Full Example: Interactive Setup Wizard

```javascript
const inquirer = require('inquirer');
const fs = require('fs');
const path = require('path');

async function setupProject() {
  console.log('🚀 Welcome to Project Setup Wizard\n');
  
  const answers = await inquirer.prompt([
    {
      type: 'input',
      name: 'projectName',
      message: 'Project name:',
      validate: input => input.length > 0 || 'Project name required'
    },
    {
      type: 'input',
      name: 'description',
      message: 'Description:',
      default: 'A new project'
    },
    {
      type: 'list',
      name: 'framework',
      message: 'Select framework:',
      choices: ['React', 'Vue', 'Angular', 'None']
    },
    {
      type: 'checkbox',
      name: 'features',
      message: 'Select features:',
      choices: [
        { name: 'TypeScript', value: 'ts' },
        { name: 'ESLint', value: 'eslint' },
        { name: 'Testing', value: 'test' },
        { name: 'CI/CD', value: 'ci' }
      ]
    },
    {
      type: 'confirm',
      name: 'createGit',
      message: 'Initialize git repository?',
      default: true
    }
  ]);
  
  console.log('\n✅ Configuration:');
  console.log(JSON.stringify(answers, null, 2));
}
```

---

## Level 4: Terminal Colors & Styling with Chalk

### Basic Colors

```javascript
const chalk = require('chalk');

console.log(chalk.red('Error message'));
console.log(chalk.green('Success!'));
console.log(chalk.yellow('Warning'));
console.log(chalk.blue('Info'));
console.log(chalk.magenta('Highlight'));
console.log(chalk.cyan('Azure'));
console.log(chalk.white('White text'));
```

### Background Colors

```javascript
console.log(chalk.red.bgWhite('Error'));
console.log(chalk.green.bgBlack('Success on black'));
console.log(chalk.blue.bgYellow('Important!'));
```

### Text Styles

```javascript
console.log(chalk.bold('Bold text'));
console.log(chalk.dim('Dimmed text'));
console.log(chalk.italic('Italic text') + chalk.reset(' (reset style)'));
console.log(chalk.underline('Underlined'));
console.log(chalk.strikethrough('Strikethrough'));
console.log(chalk.inverse('Inverted colors'));
```

### Combining Styles

```javascript
const title = chalk.bold.blue.underline('Project Setup');
const error = chalk.red.bold('ERROR:');
const success = chalk.green.bold('SUCCESS:');

console.log(title);
console.log(`${error} Something went wrong`);
console.log(`${success} Everything is fine`);
```

### RGB & Hex Colors

```javascript
console.log(chalk.rgb(255, 0, 0)('Red using RGB'));
console.log(chalk.hex('#FF00FF')('Magenta using hex'));
```

### Template Literals

```javascript
const errorCount = 5;
console.log(chalk.red`${errorCount} errors found`);

const url = 'https://example.com';
console.log(chalk.blue.underline(url));
```

### Conditional Styling

```javascript
function logStatus(status) {
  if (status === 'success') {
    console.log(chalk.green.bold('✓ Complete'));
  } else if (status === 'error') {
    console.log(chalk.red.bold('✗ Failed'));
  } else if (status === 'warn') {
    console.log(chalk.yellow('⚠ Warning'));
  } else {
    console.log(chalk.gray(`○ ${status}`));
  }
}

logStatus('success');
logStatus('error');
logStatus('warn');
logStatus('pending');
```

### Chains & Reusability

```javascript
const errorStyle = chalk.red.bold;
const successStyle = chalk.green;

console.log(errorStyle('Deployment failed'));
console.log(successStyle('Build completed'));
```

---

## Level 5: Spinners & Progress with Ora & cli-progress

### Spinner Basics (Ora)

```javascript
const ora = require('ora');

const spinner = ora('Loading data...').start();

// Simulate async operation
setTimeout(() => {
  spinner.text = 'Processing...';
  
  setTimeout(() => {
    spinner.succeed('Data loaded!');
  }, 1500);
}, 1000);
```

### Different Spinner States

```javascript
const spinner = ora('Initializing...').start();

// Success
setTimeout(() => spinner.succeed('Ready!'), 1000);

// Failure
const failSpinner = ora('Connecting...').start();
setTimeout(() => failSpinner.fail('Connection refused'), 1000);

// Warning
const warnSpinner = ora('Updating config...').start();
setTimeout(() => warnSpinner.warn('Using defaults'), 1000);

// Info
const infoSpinner = ora('Checking updates...').start();
setTimeout(() => infoSpinner.info('Up to date'), 1000);
```

### Custom Spinner

```javascript
const customSpinner = ora({
  text: 'Custom spinner',
  spinner: 'moon'  // dots, moon, line, etc.
});

customSpinner.start();
```

### Spinner with Color

```javascript
ora().start({
  text: chalk.green('Installing dependencies...'),
  color: 'green'
});
```

---

### Progress Bar (cli-progress)

```javascript
const cliProgress = require('cli-progress');

// Create a new progress bar instance
const progressBar = new cliProgress.SingleBar({
  format: 'Progress [{bar}] {percentage}% | {value}/{total} | {text}',
  barCompleteChar: '\u2588',
  barIncompleteChar: '\u2591',
  hideCursor: true
});

// Initialize with total value
const total = 100;
progressBar.start(total, 0);

// Update progress
for (let i = 0; i <= total; i++) {
  progressBar.update(i, { text: `Processing item ${i}/${total}` });
}

progressBar.stop();
```

### Multiple Progress Bars

```javascript
const { MultiBar } = require('cli-progress');

const multiBar = new MultiBar({
  format: 'Task 1: {bar} {percentage}% | Task 2: {bar2} {percentage2}%'
});

const bar1 = multiBar.createBar(100, 0);
const bar2 = multiBar.createBar(50, 0);

// Update independently
setInterval(() => {
  const val1 = bar1.value + 10;
  bar1.update(val1);
  
  if (val1 >= 100) {
    bar2.update(bar2.value + 5);
  }
}, 500);
```

### Combining Spinner & Progress

```javascript
const ora = require('ora');
const cliProgress = require('cli-progress');

async function downloadFiles(files) {
  const loading = ora('Preparing download...').start();
  
  await simulateDelay(1000);
  loading.succeed('Preparation complete');
  
  const progressBar = new cliProgress.SingleBar({
    format: 'Downloading: [{bar}] {percentage}% | {filename}'
  });
  
  progressBar.start(files.length, 0);
  
  for (let i = 0; i < files.length; i++) {
    await downloadFile(files[i]);
    progressBar.update(i + 1, { 
      filename: files[i].name 
    });
  }
  
  progressBar.stop();
  console.log('All files downloaded!');
}
```

---

## Level 6: Error Handling & Validation

### Structured Errors with Commander

```javascript
const { Command, Option } = require('commander');

program
  .command('deploy <environment>')
  .option('-c, --config <file>', 'config file path')
  .action(async (env, options) => {
    // Validate environment
    const validEnvs = ['dev', 'staging', 'prod'];
    if (!validEnvs.includes(env)) {
      console.error(chalk.red(`Invalid environment: ${env}`));
      console.log(chalk.yellow(`Valid options: ${validEnvs.join(', ')}`));
      process.exit(1);
    }
    
    try {
      await deploy(env, options.config);
    } catch (error) {
      console.error(chalk.red('Deployment failed:'), error.message);
      process.exit(1);
    }
  });
```

### Custom Error Handler

```javascript
program.exitOverride((err) => {
  if (err.code === 'commander.help') {
    // Commander's help output already sent to stdout
    return;
  }
  
  if (err.code === 'commander.unknownCommand') {
    console.error(chalk.red(`Unknown command: ${err.args.join(' ')}`));
    console.log(chalk.yellow('Run --help to see available commands'));
    process.exit(1);
  }
  
  console.error(chalk.red(err.message));
  process.exit(1);
});
```

### Validation Helpers

```javascript
function validatePort(port) {
  const num = parseInt(port, 10);
  if (isNaN(num)) return 'Must be a number';
  if (num < 1 || num > 65535) return 'Must be between 1 and 65535';
  return true;
}

function validateEmail(email) {
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  return emailRegex.test(email) || 'Invalid email format';
}

program
  .option('-p, --port <num>', 'port number', validatePort)
  .option('-e, --email <address>', 'admin email', validateEmail);
```

---

## Level 7: Configuration Files

### Loading Config from JSON

```javascript
const fs = require('fs');
const path = require('path');

function loadConfig(configPath = './config.json') {
  try {
    const absolutePath = path.resolve(configPath);
    const data = fs.readFileSync(absolutePath, 'utf8');
    return JSON.parse(data);
  } catch (error) {
    if (error.code === 'ENOENT') {
      console.warn(chalk.yellow('No config file found, using defaults'));
      return {};
    }
    throw error;
  }
}

// Usage
const config = loadConfig();
console.log('Config:', config);
```

### Config with Defaults & Environment

```javascript
const config = {
  port: 3000,
  host: 'localhost',
  env: 'development',
  ...loadConfig(),  // Override with file
  ...process.env   // Override with env vars
};

// Parse env vars
config.port = parseInt(process.env.PORT || config.port);
config.env = process.env.NODE_ENV || config.env;
```

### Hierarchical Config (multiple sources)

```javascript
const cosmiconfig = require('cosmiconfig');

async function getConfig(moduleName) {
  const searchFor = cosmiconfig(moduleName);
  const result = await searchFor.load();
  
  return result?.config || {};  // null-safe with ?.
}

const config = await getConfig('mycli');
```

---

## Level 8: File System Operations

### Create Files & Directories

```javascript
const fs = require('fs').promises;
const path = require('path');

async function createProject(projectName) {
  const dir = path.join(process.cwd(), projectName);
  
  // Create directory
  await fs.mkdir(dir, { recursive: true });
  
  // Create package.json
  const packageJson = {
    name: projectName,
    version: '1.0.0',
    scripts: { start: 'node index.js' }
  };
  
  await fs.writeFile(
    path.join(dir, 'package.json'),
    JSON.stringify(packageJson, null, 2)
  );
  
  // Create README
  await fs.writeFile(
    path.join(dir, 'README.md'),
    `# ${projectName}\n\nA new project`
  );
  
  console.log(chalk.green(`✓ Created ${projectName}/`));
}
```

### Read & Parse

```javascript
async function findAndProcessFiles(pattern) {
  const files = await fs.readdir(process.cwd());
  const matched = files.filter(f => f.includes(pattern));
  
  for (const file of matched) {
    const content = await fs.readFile(file, 'utf8');
    const data = JSON.parse(content);
    console.log(file, data.name);
  }
}
```

### Copy & Move

```javascript
const fse = require('fs-extra'); // npm install fs-extra

async function copyTemplate(src, dest) {
  await fse.copy(src, dest);
  console.log('Template copied!');
}
```

### Watch Files

```javascript
fs.watch(process.cwd(), (eventType, filename) => {
  if (filename) {
    console.log(`${eventType} on ${filename}`);
  }
});
```

---

## Level 9: External APIs & HTTP Requests

### Using axios

```javascript
const axios = require('axios'); // npm install axios

async function fetchUserData(username) {
  const spinner = ora(`Fetching ${username}...`).start();
  
  try {
    const response = await axios.get(`https://api.github.com/users/${username}`);
    spinner.succeed('User data retrieved');
    
    const user = response.data;
    console.log(`Name: ${user.name}`);
    console.log(`Repos: ${user.public_repos}`);
    console.log(`Followers: ${user.followers}`);
    
    return user;
  } catch (error) {
    spinner.fail('Failed to fetch user');
    if (error.response?.status === 404) {
      console.error(chalk.red('User not found'));
    } else {
      console.error(chalk.red(error.message));
    }
    process.exit(1);
  }
}
```

### Streaming Downloads with Progress

```javascript
const axios = require('axios');
const cliProgress = require('cli-progress');
const fs = require('fs');

async function downloadWithProgress(url, dest) {
  const writer = fs.createWriteStream(dest);
  
  const response = await axios({
    method: 'GET',
    url,
    responseType: 'stream'
  });
  
  const totalLength = response.headers['content-length'];
  const progressBar = new cliProgress.SingleBar({
    format: 'Downloading [{bar}] {percentage}% | {bytes}/{totalBytes}'
  });
  
  progressBar.start(parseInt(totalLength), 0);
  
  response.data.on('data', (chunk) => {
    progressBar.update(writer.bytesWritten);
  });
  
  response.data.pipe(writer);
  
  return new Promise((resolve, reject) => {
    writer.on('finish', () => {
      progressBar.stop();
      resolve();
    });
    writer.on('error', (err) => {
      progressBar.stop();
      reject(err);
    });
  });
}
```

---

## Level 10: Testing Your CLI

### Unit Testing with Jest

```javascript
// index.test.js
const { program } = require('./index');
const inquirer = require('inquirer');

jest.mock('inquirer');

describe('CLI Commands', () => {
  beforeEach(() => {
    program.parse = jest.fn();
    program.commands = [];
  });
  
  test('should add command', () => {
    const mockAction = jest.fn();
    
    program.command('add <file>').action(mockAction);
    
    expect(program.commands).toHaveLength(1);
  });
  
  test('should validate email input', async () => {
    inquirer.prompt.mockResolvedValue({
      email: 'test@example.com'
    });
    
    const result = await inquirer.prompt([{
      type: 'input',
      name: 'email',
      validate: (input) => {
        if (!/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(input)) {
          return 'Invalid email format';
        }
        return true;
      }
    }]);
    
    expect(result.email).toBe('test@example.com');
  });
});
```

### Integration Testing with execa

```javascript
const execa = require('execa');

test('CLI help command', async () => {
  const { stdout } = await execa('node', ['index.js', '--help']);
  expect(stdout).toContain('Usage:');
});

test('CLI create command', async () => {
  const { stdout } = await execa('node', ['index.js', 'create', 'testproject']);
  expect(stdout).toContain('Created testproject');
});
```

### E2E Testing with temp files

```javascript
const fs = require('fs');
const path = require('path');
const os = require('os');
const { exec } = require('child_process');

test('creates project with correct structure', async () => {
  const tempDir = fs.mkdtempSync(path.join(os.tmpdir(), 'test-'));
  
  try {
    exec(`node index.js create ${tempDir}/myapp`, (err) => {
      expect(err).toBeNull();
      
      const files = fs.readdirSync(tempDir + '/myapp');
      expect(files).toContain('package.json');
      expect(files).toContain('README.md');
    });
  } finally {
    fs.rmdirSync(tempDir, { recursive: true });
  }
});
```

---

## Level 11: Publishing to npm

### Prepare for Publication

1. **Create .npmignore** (exclude unnecessary files):

```
src/
*.ts
*.test.js
*.log
.DS_Store
node_modules/
```

2. **Add prepublish script** in package.json:

```json
{
  "scripts": {
    "prepublishOnly": "npm run build"
  }
}
```

3. **Tag your version**:

```bash
git tag v1.0.0
git push --tags
```

### Test publish (dry run)

```bash
npm publish --dry-run
```

### Publish

```bash
npm publish --access public  # for public packages
```

---

## Level 12: Real-World Complete Example

### Full CLI Application: `backup-cli`

```javascript
#!/usr/bin/env node

const { Command } = require('commander');
const inquirer = require('inquirer');
const chalk = require('chalk');
const ora = require('ora');
const cliProgress = require('cli-progress');
const axios = require('axios');
const fs = require('fs');
const path = require('path');
const crypto = require('crypto');

const program = new Command();

program
  .name('backup-cli')
  .description('Secure cloud backup tool')
  .version('1.0.0');

// Config command
program.command('config')
  .description('Configure backup settings')
  .action(async () => {
    const answers = await inquirer.prompt([
      {
        type: 'input',
        name: 'apiKey',
        message: 'Enter your API key:',
        validate: input => input.length > 0 || 'API key required'
      },
      {
        type: 'input',
        name: 'remotePath',
        message: 'Remote backup path:',
        default: '/backups'
      },
      {
        type: 'confirm',
        name: 'encrypt',
        message: 'Encrypt backups?',
        default: true
      }
    ]);
    
    const configPath = path.join(process.cwd(), '.backuprc.json');
    fs.writeFileSync(configPath, JSON.stringify(answers, null, 2));
    console.log(chalk.green('✓ Configuration saved!'));
  });

// Backup command
program.command('backup [files...]')
  .description('Backup files to cloud')
  .option('-e, --encrypt', 'Encrypt before upload')
  .option('-c, --compress', 'Compress files')
  .option('--dry-run', 'Simulate without uploading')
  .action(async (files, options) => {
    // Load config
    const configPath = path.join(process.cwd(), '.backuprc.json');
    if (!fs.existsSync(configPath)) {
      console.error(chalk.red('Configuration not found! Run: backup config'));
      process.exit(1);
    }
    
    const config = JSON.parse(fs.readFileSync(configPath, 'utf8'));
    
    // Determine files to backup
    const filesToBackup = files.length > 0 
      ? files 
      : getModifiedFiles(); // custom function
    
    if (filesToBackup.length === 0) {
      console.log(chalk.yellow('No files to backup'));
      return;
    }
    
    console.log(chalk.bold.blue(`\n📦 Backing up ${filesToBackup.length} files\n`));
    
    // Process files with progress
    const spinner = ora('Preparing...').start();
    const progressBar = new cliProgress.SingleBar({
      format: 'Processing [{bar}] {percentage}% | {filename}',
      barCompleteChar: '\u2588',
      barIncompleteChar: '\u2591'
    });
    
    progressBar.start(filesToBackup.length, 0);
    
    try {
      for (let i = 0; i < filesToBackup.length; i++) {
        const file = filesToBackup[i];
        spinner.text = `Processing ${path.basename(file)}`;
        
        // Simulate processing
        await processFile(file, options);
        
        progressBar.update(i + 1, { filename: path.basename(file) });
      }
      
      spinner.succeed('All files processed!');
      
      if (!options.dryRun) {
        const uploadSpinner = ora('Uploading to cloud...').start();
        await uploadFiles(config, filesToBackup);
        uploadSpinner.succeed('Upload complete!');
      }
      
      console.log(chalk.green.bold('\n✅ Backup completed successfully!'));
      
    } catch (error) {
      spinner.fail('Backup failed');
      console.error(chalk.red(error.message));
      process.exit(1);
    } finally {
      progressBar.stop();
    }
  });

// Restore command
program.command('restore <backupId>')
  .description('Restore files from backup')
  .option('-o, --output <dir>', 'output directory', './restored')
  .action(async (backupId, options) => {
    const spinner = ora('Fetching backup list...').start();
    
    try {
      const backups = await listBackups(); // fetch from API
      const target = backups.find(b => b.id === backupId);
      
      if (!target) {
        throw new Error(`Backup ${backupId} not found`);
      }
      
      spinner.succeed(`Found backup from ${target.date}`);
      
      const progressBar = new cliProgress.SingleBar({
        format: 'Restoring [{bar}] {percentage}%'
      });
      
      progressBar.start(100, 0);
      
      await restoreBackup(target, options.output, (percent) => {
        progressBar.update(percent);
      });
      
      progressBar.stop();
      spinner.succeed('Restore complete!');
      
    } catch (error) {
      spinner.fail('Restore failed');
      console.error(chalk.red(error.message));
    }
  });

// Stats command
program.command('stats')
  .description('Show backup statistics')
  .option('-j, --json', 'Output as JSON')
  .action(async (options) => {
    const stats = await getStats();
    
    if (options.json) {
      console.log(JSON.stringify(stats, null, 2));
    } else {
      console.log(chalk.bold('\n📊 Backup Statistics\n'));
      console.log(`Total backups: ${chalk.cyan(stats.total)}`);
      console.log(`Total size: ${chalk.yellow(formatBytes(stats.size))}`);
      console.log(`Last backup: ${chalk.green(stats.lastBackup)}`);
    }
  });

// Parse arguments
program.parse();

// Helper functions (would be implemented separately)
function getModifiedFiles() {
  // Return list of modified files
  return [];
}

async function processFile(file, options) {
  // Process single file
  await new Promise(resolve => setTimeout(resolve, 100));
}

async function uploadFiles(config, files) {
  // Upload to cloud
  await new Promise(resolve => setTimeout(resolve, 2000));
}

async function listBackups() {
  return [];
}

async function restoreBackup(backup, output, onProgress) {
  for (let i = 0; i <= 100; i += 10) {
    await new Promise(resolve => setTimeout(resolve, 100));
    onProgress(i);
  }
}

async function getStats() {
  return { total: 5, size: 10485760, lastBackup: '2024-01-15' };
}

function formatBytes(bytes) {
  const units = ['B', 'KB', 'MB', 'GB'];
  let size = bytes;
  let unitIndex = 0;
  
  while (size >= 1024 && unitIndex < units.length - 1) {
    size /= 1024;
    unitIndex++;
  }
  
  return `${size.toFixed(1)} ${units[unitIndex]}`;
}
```

---

## Level 13: Advanced Patterns

### Middleware Pattern

```javascript
// Apply middleware to commands
program
  .command('deploy')
  .middleware(async (args, command, next) => {
    // Pre-processing
    console.log('Checking permissions...');
    
    if (!hasPermission()) {
      throw new Error('Insufficient permissions');
    }
    
    await next(); // Continue to command handler
  })
  .action(() => {
    // Actual command
  });
```

### Plugin System

```javascript
class CLI {
  constructor() {
    this.plugins = [];
  }
  
  use(plugin) {
    plugin.install(this);
    this.plugins.push(plugin);
  }
  
  async run(args) {
    for (const plugin of this.plugins) {
      if (plugin.beforeRun) {
        await plugin.beforeRun(args);
      }
    }
    
    // Execute command
    
    for (const plugin of this.plugins) {
      if (plugin.afterRun) {
        await plugin.afterRun(args);
      }
    }
  }
}
```

### Logging Levels

```javascript
const logLevels = {
  error: 0,
  warn: 1,
  info: 2,
  debug: 3
};

const currentLevel = process.env.DEBUG ? logLevels.debug : logLevels.info;

function log(level, message, style = {}) {
  if (logLevels[level] > currentLevel) return;
  
  let styled = message;
  switch(level) {
    case 'error': styled = chalk.red(`✗ ${message}`); break;
    case 'warn': styled = chalk.yellow(`⚠ ${message}`); break;
    case 'info': styled = chalk.blue(`ℹ ${message}`); break;
    case 'debug': styled = chalk.gray(`🐛 ${message}`); break;
  }
  
  console.log(styled);
}

// Usage
log('info', 'Starting process');
log('debug', 'Debug details', { verbose: true });
```

---

## Level 14: Performance Tips

1. **Lazy load modules** - Require inside functions
2. **Cache expensive operations** - Use memory or disk cache
3. **Stream large files** - Don't load into memory
4. **Use native modules** where possible (fs, path, crypto)
5. **Profile with Node's inspector** for bottlenecks

---

## Common Gotchas

1. **Shebang line must be first**: `#!/usr/bin/env node` at absolute top
2. **Make files executable** on Unix: `chmod +x bin/cli.js`
3. **Parse early**: `program.parse(process.argv)` not `program.parse()`
4. **Escape special characters** in regex patterns
5. **Handle SIGINT** for clean exits: `process.on('SIGINT', () => process.exit(0));`

---

## Resources

- [Commander.js Docs](https://commander.js.org/)
- [Inquirer.js Docs](https://github.com/SBoudrias/Inquirer.js/)
- [Chalk Docs](https://www.npmjs.com/package/chalk)
- [Ora Docs](https://github.com/sindresorhus/ora)
- [cli-progress Docs](https://github.com/npkgz/cli-progress)
- [Awesome CLI Tools](https://github.com/ahmadawais/awesome-cli)

---

## Next Steps

1. Build a simple utility (todo-list manager, file organizer)
2. Add config management + persistence
3. Add plugin architecture
4. Create tests and CI/CD pipeline
5. Publish to npm!