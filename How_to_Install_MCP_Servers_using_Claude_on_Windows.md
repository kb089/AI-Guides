# 🌟 Complete Guide to Setting Up Claude Desktop with MCP Servers
## Starting from a Fresh Windows Install

### What We'll Install:
1. Claude Desktop
2. VS Code (a text editor)
3. Node.js (required for MCP servers)
4. Your first MCP server!

### Part 1: Getting Started with Claude 🤖
1. Create a Claude.ai account:
   - Go to https://claude.ai
   - Click "Sign up"
   - Enter your email address
   - Create a password
   - Verify your email
   - Complete any additional verification steps

2. Install Claude Desktop:
   - Go to https://claude.ai/desktop
   - Click the "Download for Windows" button
   - When the installer downloads (ClaudeSetup.exe), click on it
   - If you see a Windows security warning, click "More info" then "Run anyway"
   - Follow the installer steps:
     - Choose "Install for all users" (recommended)
     - Leave the installation location as default
     - Click "Install"
     - Wait for installation to complete
     - Click "Finish"

3. Sign in to Claude Desktop:
   - When Claude Desktop starts, click "Sign in with Claude.ai"
   - Use the email and password you created in step 1
   - You might need to verify your email again

### Finding Your Windows Username 👤
You'll need this later! Here's how to find it:
1. Press Windows key + R
2. Type "cmd" and press Enter
3. In the black window, type: `echo %USERNAME%`
4. The text that appears is your username
5. Write this down! You'll need it later

### Part 2: Installing VS Code 📝
1. Go to https://code.visualstudio.com
2. Click the big blue "Download for Windows" button
3. When the installer downloads, click on it
4. In the installer:
   - Accept the license agreement
   - Leave the installation location as default
   - Check all these boxes:
     ☑️ Create a desktop icon
     ☑️ Add "Open with Code" action
     ☑️ Register Code as an editor
     ☑️ Add to PATH
   - Click "Install"
   - Click "Finish"

### Part 3: Installing Node.js ⚙️
1. Go to https://nodejs.org
2. Click the big green button that says "LTS"
   ```
   [Download LTS]  [Current]
   v18.x.x        v20.x.x
   Recommended    Latest Features
   ```
3. When the installer (node-v18.x.x-x64.msi) downloads, click on it
4. In the installer:
   - Accept the license agreement → Next
   - Leave the destination folder as default → Next
   - In "Custom Setup", leave all features checked:
     ```
     [X] Node.js runtime
     [X] npm package manager
     [X] Online documentation shortcuts
     [X] Add to PATH
     ```
   - Click "Next"
   - Check "Automatically install necessary tools" → Next
   - Click "Install"
   - If you see a Windows security prompt about PowerShell, click "Yes"
   - Wait for both installations to finish
   - Click "Finish"

5. Let's verify Node.js installed correctly:
   - Press Windows key + R
   - Type "cmd" and press Enter
   - In the black window that opens, type:
     ```
     node --version
     ```
   - You should see something like "v18.x.x"
   - Then type:
     ```
     npm --version
     ```
   - You should see something like "9.x.x"
   - If you see both version numbers, Node.js is installed correctly!

### Part 4: Setting Up VS Code for MCP Work 🛠️
1. Open VS Code
2. Set up PowerShell (we need this for MCP servers):
   - Press `Ctrl+Shift+P`
   - Type "Terminal: Select Default Profile"
   - Click on "PowerShell"
   - Press `Ctrl+Shift+P` again
   - Type "Developer: Reload Window" and click it

3. Open PowerShell in VS Code:
   - Press Ctrl + ` (the key above Tab)
   - You should see a blue PowerShell prompt like this:
     ```
     PS C:\Users\YourUsername>
     ```
   - Type this to check PowerShell:
     ```
     $PSVersionTable.PSVersion
     ```
   - You should see version information

### Part 5: Setting Up MCP Servers Directory 📁
1. Create a folder for all your MCP servers:
   - Press Windows key + E to open File Explorer
   - Click on "Documents" on the left
   - Right-click in the empty space → New → Folder
   - Name it "MCP_Servers"

2. Create the Claude config folder:
   - In File Explorer, click the address bar
   - Copy and paste this path:
     ```
     %LocalAppData%\AnthropicClaude\config
     ```
   - Press Enter
   - If the 'config' folder doesn't exist:
     - Right-click → New → Folder
     - Name it "config"

3. Create the config file:
   - Open VS Code
   - Click File → Open Folder
   - Navigate to:
     ```
     C:\Users\YOUR_USERNAME\AppData\Local\AnthropicClaude\config
     ```
   - Click "Select Folder"
   - Click File → New File
   - Name it "claude_desktop_config.json"
   - Paste this basic configuration:
     ```json
     {
       "mcpServers": {}
     }
     ```
   - Press Ctrl + S to save

### Part 6: Installing Your First MCP Server 🚀
Let's install a simple "hello" server to test everything:

1. Download the server:
   - Go to your Documents\MCP_Servers folder
   - Create a new folder called "hello-server"
   - Open VS Code
   - Click File → Open Folder
   - Select the hello-server folder
   - Create two new files:

2. Create package.json:
   ```json
   {
     "name": "hello-server",
     "version": "1.0.0",
     "type": "module",
     "main": "index.js",
     "dependencies": {
       "@modelcontextprotocol/sdk": "latest"
     }
   }
   ```

3. Create index.js:
   ```javascript
   #!/usr/bin/env node
   import { Server } from '@modelcontextprotocol/sdk/server/index.js';
   import { StdioServerTransport } from '@modelcontextprotocol/sdk/server/stdio.js';
   import { CallToolRequestSchema, ListToolsRequestSchema } from '@modelcontextprotocol/sdk/types.js';

   const server = new Server(
     {
       name: 'hello-server',
       version: '0.1.0',
     },
     {
       capabilities: {
         tools: {},
       },
     }
   );

   server.setRequestHandler(ListToolsRequestSchema, async () => ({
     tools: [
       {
         name: 'say_hello',
         description: 'Say hello to someone',
         inputSchema: {
           type: 'object',
           properties: {
             name: {
               type: 'string',
               description: 'Name to say hello to',
             },
           },
           required: ['name'],
         },
       },
     ],
   }));

   server.setRequestHandler(CallToolRequestSchema, async (request) => {
     if (request.params.name === 'say_hello') {
       const name = request.params.arguments.name;
       return {
         content: [
           {
             type: 'text',
             text: `Hello, ${name}!`,
           },
         ],
       };
     }
     return {
       content: [
         {
           type: 'text',
           text: 'Unknown command',
         },
       ],
       isError: true,
     };
   });

   const transport = new StdioServerTransport();
   server.connect(transport).catch(console.error);
   ```

4. Install the server:
   - Open PowerShell in VS Code (Ctrl + `)
   - Type:
     ```
     npm install
     ```
   - Wait for it to finish

5. Update Claude's config:
   - Open C:\Users\YOUR_USERNAME\AppData\Local\AnthropicClaude\config\claude_desktop_config.json
   - Replace its contents with:
     ```json
     {
       "mcpServers": {
         "hello": {
           "command": "node",
           "args": ["C:/Users/YOUR_USERNAME/Documents/MCP_Servers/hello-server/index.js"],
           "disabled": false,
           "autoApprove": []
         }
       }
     }
     ```
   - Replace YOUR_USERNAME with your actual Windows username (the one you wrote down earlier!)
   - Save the file (Ctrl + S)

6. Start using your server:
   - Close Claude Desktop completely:
     - Find the Claude icon in your system tray (bottom right)
     - Right-click it
     - Click "Quit"
   - Start Claude Desktop again
   - Try it out! Type: "say hello to Bob"
   - Claude should respond with "Hello, Bob!"

### Common Problems and Solutions 🔧
1. "Command not found" error:
   - Make sure Node.js is installed correctly
   - Open cmd and type `node --version`
   - If it doesn't work, try uninstalling and reinstalling Node.js

2. "Cannot find module" error:
   - Make sure you're in the right folder in VS Code
   - Try running `npm install` again
   - Check if your package.json file is correct

3. Claude can't see the server:
   - Check your username in the config file path
   - Make sure you fully closed Claude (right-click → Quit)
   - Check that all file paths use forward slashes (/)

4. PowerShell isn't working:
   - Press Ctrl+Shift+P in VS Code
   - Type "Terminal: Select Default Profile"
   - Choose PowerShell
   - Reload VS Code

### Need Help? 🆘
If you get stuck:
1. Check the steps again carefully
2. Make sure all file paths match your username
3. Ask Claude! Just say what step you're stuck on and what error you're seeing

Remember: You can't break anything by trying! If something goes wrong, we can always start over. Have fun with your MCP servers! 🚀
