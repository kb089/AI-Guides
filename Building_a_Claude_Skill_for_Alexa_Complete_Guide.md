# Building a Claude Skill for Alexa: Complete Guide

## Table of Contents
1. [Prerequisites](#prerequisites)
2. [Setup and Configuration](#setup-and-configuration)
3. [Creating the Alexa Skill](#creating-the-alexa-skill)
4. [Integrating Claude's API](#integrating-claudes-api)
5. [Designing the Conversation Flow](#designing-the-conversation-flow)
6. [Handling Context and Memory](#handling-context-and-memory)
7. [Testing Your Skill](#testing-your-skill)
8. [Deployment and Certification](#deployment-and-certification)
9. [Monitoring and Maintenance](#monitoring-and-maintenance)
10. [Advanced Features](#advanced-features)
11. [Troubleshooting](#troubleshooting)
12. [Example Implementation](#example-implementation)
13. [Conclusion](#conclusion)
14. [Further Reading](#further-reading)

## Prerequisites

### 1. Accounts and Subscriptions
- **Amazon Developer Account**: Create one at [developer.amazon.com](https://developer.amazon.com)
- **AWS Account**: Sign up at [aws.amazon.com](https://aws.amazon.com)
- **Anthropic API Key**: Get access to the Claude API at [anthropic.com](https://www.anthropic.com)

### 2. Technical Requirements
- **Programming Knowledge**: Familiarity with JavaScript/Node.js or Python
- **Development Environment**:
  - Code editor (VSCode, Sublime Text, etc.)
  - Node.js (version 14.x or higher) or Python (3.7+) installed
  - npm or pip package manager
- **Optional**:
  - AWS CLI installed and configured
  - Git for version control

### 3. Hardware for Testing
- Amazon Echo device or the Alexa mobile app
- Computer with internet connection

## Setup and Configuration

### 1. Set Up Your Development Environment

#### Install Node.js and npm (if using JavaScript)
```bash
# For macOS using Homebrew
brew install node

# For Ubuntu/Debian
sudo apt update
sudo apt install nodejs npm

# For Windows
# Download installer from https://nodejs.org/
```

#### Install Python and pip (if using Python)
```bash
# For macOS using Homebrew
brew install python

# For Ubuntu/Debian
sudo apt update
sudo apt install python3 python3-pip

# For Windows
# Download installer from https://www.python.org/
```

#### Install the Alexa Skills Kit (ASK) CLI
```bash
npm install -g ask-cli
ask configure
```

### 2. Configure Your AWS Account for Lambda Functions

1. Log in to your AWS Console
2. Navigate to IAM (Identity and Access Management)
3. Create a new IAM role with the following permissions:
   - AWSLambdaBasicExecutionRole
   - AmazonDynamoDBFullAccess (if using DynamoDB for session storage)
   - AmazonS3ReadOnlyAccess (if using S3 for assets)

### 3. Set Up Your Anthropic API Credentials

1. Log in to your Anthropic account
2. Navigate to the API section
3. Generate or retrieve your API key
4. Note down the API key for later use

## Creating the Alexa Skill

### 1. Create a New Alexa Skill

#### Using ASK CLI
```bash
ask new
# Follow the prompts to set up your skill
```

#### Using the Alexa Developer Console
1. Go to [developer.amazon.com](https://developer.amazon.com)
2. Navigate to the Alexa Skills Kit
3. Click "Create Skill"
4. Enter a skill name (e.g., "Claude Assistant")
5. Select "Custom" model
6. Choose a hosting method (AWS Lambda recommended)
7. Select "Start from scratch" as your template

### 2. Define Your Skill's Interaction Model

#### Define Invocation Name
Set a name that users will say to invoke your skill:
```json
{
  "interactionModel": {
    "languageModel": {
      "invocationName": "claude assistant",
      "...": "..."
    }
  }
}
```

#### Define Intents
Create the following intents:
1. **AskClaudeIntent**: Main intent for asking Claude questions
2. **AMAZON.HelpIntent**: For providing help
3. **AMAZON.StopIntent**: For stopping the skill
4. **AMAZON.CancelIntent**: For canceling actions

#### Sample Intent JSON
```json
{
  "intents": [
    {
      "name": "AskClaudeIntent",
      "slots": [
        {
          "name": "Query",
          "type": "AMAZON.SearchQuery"
        }
      ],
      "samples": [
        "ask {Query}",
        "tell me {Query}",
        "I want to know {Query}",
        "can you explain {Query}"
      ]
    },
    {
      "name": "AMAZON.HelpIntent",
      "samples": []
    },
    {
      "name": "AMAZON.StopIntent",
      "samples": []
    },
    {
      "name": "AMAZON.CancelIntent",
      "samples": []
    }
  ]
}
```

### 3. Set Up Your Backend Function

#### Create an AWS Lambda Function
1. Go to the AWS Lambda Console
2. Click "Create function"
3. Select "Author from scratch"
4. Enter a function name
5. Select Node.js 14.x or Python 3.8 as the runtime
6. Choose the IAM role you created earlier
7. Click "Create function"

#### Configure Function Settings
1. Set the handler file and method (e.g., `index.handler` for Node.js)
2. Set environment variables:
   - `ANTHROPIC_API_KEY`: Your Claude API key
   - `MAX_RESPONSE_LENGTH`: Maximum length for Alexa responses (e.g., 8000)
   - `MODEL_NAME`: The Claude model to use (e.g., `claude-3-7-sonnet-20250219`)

## Integrating Claude's API

### 1. Install Required Dependencies

#### For Node.js/JavaScript
```bash
npm init -y
npm install ask-sdk-core axios
```

#### For Python
```bash
pip install ask-sdk anthropic
```

### 2. Set Up the Anthropic Client

#### Node.js Implementation
```javascript
const axios = require('axios');

const anthropicClient = {
  async generateResponse(prompt, conversationHistory = []) {
    try {
      const response = await axios.post(
        'https://api.anthropic.com/v1/messages',
        {
          model: process.env.MODEL_NAME,
          messages: [...conversationHistory, { role: 'user', content: prompt }],
          max_tokens: 1024
        },
        {
          headers: {
            'Content-Type': 'application/json',
            'x-api-key': process.env.ANTHROPIC_API_KEY,
            'anthropic-version': '2023-06-01'
          }
        }
      );
      
      return response.data.content[0].text;
    } catch (error) {
      console.error('Error calling Anthropic API:', error.response ? error.response.data : error.message);
      return "I'm having trouble connecting to my systems right now. Please try again later.";
    }
  }
};

module.exports = anthropicClient;
```

#### Python Implementation
```python
import os
from anthropic import Anthropic

class AnthropicClient:
    def __init__(self):
        self.client = Anthropic(api_key=os.environ.get("ANTHROPIC_API_KEY"))
        self.model = os.environ.get("MODEL_NAME", "claude-3-7-sonnet-20250219")
    
    def generate_response(self, prompt, conversation_history=None):
        if conversation_history is None:
            conversation_history = []
        
        try:
            messages = conversation_history + [{"role": "user", "content": prompt}]
            response = self.client.messages.create(
                model=self.model,
                messages=messages,
                max_tokens=1024
            )
            return response.content[0].text
        except Exception as e:
            print(f"Error calling Anthropic API: {str(e)}")
            return "I'm having trouble connecting to my systems right now. Please try again later."
```

### 3. Create the Alexa Skill Handler

#### Node.js Implementation
```javascript
const Alexa = require('ask-sdk-core');
const anthropicClient = require('./anthropic-client');

// Launch request handler
const LaunchRequestHandler = {
  canHandle(handlerInput) {
    return Alexa.getRequestType(handlerInput.requestEnvelope) === 'LaunchRequest';
  },
  handle(handlerInput) {
    const speakOutput = 'Welcome to Claude Assistant. What would you like to know?';
    
    return handlerInput.responseBuilder
      .speak(speakOutput)
      .reprompt(speakOutput)
      .getResponse();
  }
};

// Ask Claude Intent handler
const AskClaudeIntentHandler = {
  canHandle(handlerInput) {
    return Alexa.getRequestType(handlerInput.requestEnvelope) === 'IntentRequest'
      && Alexa.getIntentName(handlerInput.requestEnvelope) === 'AskClaudeIntent';
  },
  async handle(handlerInput) {
    const query = Alexa.getSlotValue(handlerInput.requestEnvelope, 'Query');
    
    // Get session attributes for conversation history
    const sessionAttributes = handlerInput.attributesManager.getSessionAttributes();
    if (!sessionAttributes.conversationHistory) {
      sessionAttributes.conversationHistory = [];
    }
    
    // Set a system prompt for Claude
    const systemPrompt = "You are Claude, an AI assistant created by Anthropic, now available through an Alexa Skill. Keep your responses concise and suitable for voice. Avoid lengthy lists or complex explanations unless specifically requested. Avoid mentioning URLs or visual elements.";
    
    // Format the query with the system prompt
    const formattedQuery = `${systemPrompt}\n\nUser query: ${query}`;
    
    // Call Claude API
    let claudeResponse = await anthropicClient.generateResponse(formattedQuery, sessionAttributes.conversationHistory);
    
    // Truncate response if needed for Alexa
    if (claudeResponse.length > 8000) {
      claudeResponse = claudeResponse.substring(0, 7997) + '...';
    }
    
    // Update conversation history
    sessionAttributes.conversationHistory.push(
      { role: 'user', content: query },
      { role: 'assistant', content: claudeResponse }
    );
    
    // Keep only last 10 messages to manage context window
    if (sessionAttributes.conversationHistory.length > 10) {
      sessionAttributes.conversationHistory = sessionAttributes.conversationHistory.slice(-10);
    }
    
    handlerInput.attributesManager.setSessionAttributes(sessionAttributes);
    
    return handlerInput.responseBuilder
      .speak(claudeResponse)
      .reprompt('Is there anything else you would like to know?')
      .getResponse();
  }
};

// Help Intent handler
const HelpIntentHandler = {
  canHandle(handlerInput) {
    return Alexa.getRequestType(handlerInput.requestEnvelope) === 'IntentRequest'
      && Alexa.getIntentName(handlerInput.requestEnvelope) === 'AMAZON.HelpIntent';
  },
  handle(handlerInput) {
    const speakOutput = 'You can ask me any question and I\'ll use Claude\'s intelligence to answer. For example, try saying "ask Claude what is quantum computing?"';
    
    return handlerInput.responseBuilder
      .speak(speakOutput)
      .reprompt(speakOutput)
      .getResponse();
  }
};

// Cancel and Stop Intent handler
const CancelAndStopIntentHandler = {
  canHandle(handlerInput) {
    return Alexa.getRequestType(handlerInput.requestEnvelope) === 'IntentRequest'
      && (Alexa.getIntentName(handlerInput.requestEnvelope) === 'AMAZON.CancelIntent'
          || Alexa.getIntentName(handlerInput.requestEnvelope) === 'AMAZON.StopIntent');
  },
  handle(handlerInput) {
    const speakOutput = 'Goodbye!';
    
    return handlerInput.responseBuilder
      .speak(speakOutput)
      .getResponse();
  }
};

// Error handler
const ErrorHandler = {
  canHandle() {
    return true;
  },
  handle(handlerInput, error) {
    console.log(`Error handled: ${error.message}`);
    
    const speakOutput = 'Sorry, I had trouble understanding your request. Please try again.';
    
    return handlerInput.responseBuilder
      .speak(speakOutput)
      .reprompt(speakOutput)
      .getResponse();
  }
};

// Session Ended Request handler
const SessionEndedRequestHandler = {
  canHandle(handlerInput) {
    return Alexa.getRequestType(handlerInput.requestEnvelope) === 'SessionEndedRequest';
  },
  handle(handlerInput) {
    console.log(`Session ended with reason: ${handlerInput.requestEnvelope.request.reason}`);
    return handlerInput.responseBuilder.getResponse();
  }
};

// Export handler function
exports.handler = Alexa.SkillBuilders.custom()
  .addRequestHandlers(
    LaunchRequestHandler,
    AskClaudeIntentHandler,
    HelpIntentHandler,
    CancelAndStopIntentHandler,
    SessionEndedRequestHandler
  )
  .addErrorHandlers(ErrorHandler)
  .lambda();
```

#### Python Implementation
```python
import logging
import ask_sdk_core.utils as ask_utils
from ask_sdk_core.skill_builder import SkillBuilder
from ask_sdk_core.dispatch_components import AbstractRequestHandler
from ask_sdk_core.dispatch_components import AbstractExceptionHandler
from ask_sdk_core.handler_input import HandlerInput
from ask_sdk_model import Response
from anthropic_client import AnthropicClient

logger = logging.getLogger(__name__)
logger.setLevel(logging.INFO)

anthropic_client = AnthropicClient()

class LaunchRequestHandler(AbstractRequestHandler):
    """Handler for Skill Launch."""
    def can_handle(self, handler_input):
        return ask_utils.is_request_type("LaunchRequest")(handler_input)

    def handle(self, handler_input):
        speak_output = "Welcome to Claude Assistant. What would you like to know?"

        return (
            handler_input.response_builder
                .speak(speak_output)
                .ask(speak_output)
                .response
        )


class AskClaudeIntentHandler(AbstractRequestHandler):
    """Handler for Ask Claude Intent."""
    def can_handle(self, handler_input):
        return (ask_utils.is_request_type("IntentRequest")(handler_input) and
                ask_utils.is_intent_name("AskClaudeIntent")(handler_input))

    def handle(self, handler_input):
        # Get the query from the slot
        query = ask_utils.get_slot_value(handler_input, "Query")
        
        # Get session attributes for conversation history
        session_attr = handler_input.attributes_manager.session_attributes
        if "conversation_history" not in session_attr:
            session_attr["conversation_history"] = []
        
        # Set a system prompt for Claude
        system_prompt = "You are Claude, an AI assistant created by Anthropic, now available through an Alexa Skill. Keep your responses concise and suitable for voice. Avoid lengthy lists or complex explanations unless specifically requested. Avoid mentioning URLs or visual elements."
        
        # Format the query with the system prompt
        formatted_query = f"{system_prompt}\n\nUser query: {query}"
        
        # Call Claude API
        claude_response = anthropic_client.generate_response(
            formatted_query, 
            session_attr["conversation_history"]
        )
        
        # Truncate response if needed for Alexa
        if len(claude_response) > 8000:
            claude_response = claude_response[:7997] + "..."
        
        # Update conversation history
        session_attr["conversation_history"].extend([
            {"role": "user", "content": query},
            {"role": "assistant", "content": claude_response}
        ])
        
        # Keep only last 10 messages to manage context window
        if len(session_attr["conversation_history"]) > 10:
            session_attr["conversation_history"] = session_attr["conversation_history"][-10:]
        
        handler_input.attributes_manager.session_attributes = session_attr
        
        return (
            handler_input.response_builder
                .speak(claude_response)
                .ask("Is there anything else you would like to know?")
                .response
        )


class HelpIntentHandler(AbstractRequestHandler):
    """Handler for Help Intent."""
    def can_handle(self, handler_input):
        return (ask_utils.is_request_type("IntentRequest")(handler_input) and
                ask_utils.is_intent_name("AMAZON.HelpIntent")(handler_input))

    def handle(self, handler_input):
        speak_output = "You can ask me any question and I'll use Claude's intelligence to answer. For example, try saying 'ask Claude what is quantum computing?'"

        return (
            handler_input.response_builder
                .speak(speak_output)
                .ask(speak_output)
                .response
        )


class CancelOrStopIntentHandler(AbstractRequestHandler):
    """Handler for Cancel or Stop Intent."""
    def can_handle(self, handler_input):
        return (ask_utils.is_request_type("IntentRequest")(handler_input) and
                (ask_utils.is_intent_name("AMAZON.CancelIntent")(handler_input) or
                 ask_utils.is_intent_name("AMAZON.StopIntent")(handler_input)))

    def handle(self, handler_input):
        speak_output = "Goodbye!"

        return (
            handler_input.response_builder
                .speak(speak_output)
                .response
        )


class SessionEndedRequestHandler(AbstractRequestHandler):
    """Handler for Session End."""
    def can_handle(self, handler_input):
        return ask_utils.is_request_type("SessionEndedRequest")(handler_input)

    def handle(self, handler_input):
        # Any cleanup logic goes here
        return handler_input.response_builder.response


class CatchAllExceptionHandler(AbstractExceptionHandler):
    """Global exception handler"""
    def can_handle(self, handler_input, exception):
        return True

    def handle(self, handler_input, exception):
        logger.error(exception, exc_info=True)
        speak_output = "Sorry, I had trouble understanding your request. Please try again."

        return (
            handler_input.response_builder
                .speak(speak_output)
                .ask(speak_output)
                .response
        )


# The SkillBuilder object acts as the entry point for your skill
sb = SkillBuilder()

# Register request handlers
sb.add_request_handler(LaunchRequestHandler())
sb.add_request_handler(AskClaudeIntentHandler())
sb.add_request_handler(HelpIntentHandler())
sb.add_request_handler(CancelOrStopIntentHandler())
sb.add_request_handler(SessionEndedRequestHandler())

# Register exception handlers
sb.add_exception_handler(CatchAllExceptionHandler())

# Lambda handler
lambda_handler = sb.lambda_handler()
```

## Designing the Conversation Flow

### 1. Define the User Experience Flow

1. **Invocation**: User says "Alexa, ask Claude Assistant about [topic]"
2. **Processing**: 
   - Alexa captures the query
   - Sends it to your Lambda function
   - Lambda calls the Claude API
   - Claude generates a response
   - Lambda formats and returns the response to Alexa
3. **Response**: Alexa speaks Claude's response back to the user
4. **Follow-up**: Alexa asks if there's anything else the user would like to know

### 2. Optimize for Voice Interaction

#### Response Formatting Guidelines
1. Keep responses concise (under 30 seconds of speaking time)
2. Break long responses into chunks if necessary
3. Remove markdown or formatting that doesn't translate to voice
4. Add speech pauses (`<break/>` SSML tags) where appropriate
5. Use simpler language for better comprehension

#### Implement SSML Enhancements
```javascript
// Example of adding SSML to Claude's response
function addSSML(text) {
  // Replace bullet points with pauses
  text = text.replace(/•/g, '<break time="300ms"/>');
  
  // Add pauses after periods
  text = text.replace(/\. /g, '.<break time="200ms"/> ');
  
  // Wrap in SSML tags
  return `<speak>${text}</speak>`;
}

// Then in your intent handler:
return handlerInput.responseBuilder
  .speak(addSSML(claudeResponse))
  .reprompt('Is there anything else you would like to know?')
  .getResponse();
```

## Handling Context and Memory

### 1. Maintain Conversation Context

#### Using Session Attributes
```javascript
// Store conversation history in session attributes
const sessionAttributes = handlerInput.attributesManager.getSessionAttributes();
if (!sessionAttributes.conversationHistory) {
  sessionAttributes.conversationHistory = [];
}

// Update after each interaction
sessionAttributes.conversationHistory.push(
  { role: 'user', content: query },
  { role: 'assistant', content: claudeResponse }
);

// Save back to the session
handlerInput.attributesManager.setSessionAttributes(sessionAttributes);
```

### 2. Implement Persistent Storage (Optional)

#### Using DynamoDB
```javascript
const persistenceAdapter = require('ask-sdk-dynamodb-persistence-adapter');

// Configure the adapter
const persistenceAdapter = new persistenceAdapter.DynamoDbPersistenceAdapter({
  tableName: 'ClaudeAlexaConversations',
  createTable: true,
  dynamoDBClient: new AWS.DynamoDB({apiVersion: 'latest', region: 'us-east-1'})
});

// Add to skill builder
exports.handler = Alexa.SkillBuilders.custom()
  .addRequestHandlers(/* ... */)
  .addErrorHandlers(/* ... */)
  .withPersistenceAdapter(persistenceAdapter)
  .lambda();
```

## Testing Your Skill

### 1. Local Testing

#### Using ASK CLI
```bash
ask dialog --locale en-US
> open claude assistant
> ask what is quantum computing
```

#### Using Bespoken Tools
```bash
npm install -g bespoken-tools
bst proxy lambda index.js
bst speak open claude assistant
bst speak ask what is quantum computing
```

### 2. Test in the Alexa Developer Console

1. Navigate to your skill in the Alexa Developer Console
2. Go to the "Test" tab
3. Enable testing for your skill
4. Use the text input or microphone to test commands

### 3. Testing on an Alexa Device

1. Make sure your skill is in "Development" mode
2. Log in to the Alexa app with the same account as your developer account
3. Enable the skill in the app
4. Test on your Echo device

## Deployment and Certification

### 1. Finalize Your Skill

1. Complete all testing
2. Ensure your skill works consistently
3. Review all error handling

### 2. Prepare for Submission

1. Create a skill icon (108x108px and 512x512px)
2. Write skill descriptions:
   - Short description (160 characters)
   - Full description (4000 characters)
   - Example phrases (3 required)
3. Select appropriate categories
4. Define appropriate age ratings

### 3. Submit for Certification

1. Navigate to the "Certification" tab in the Alexa Developer Console
2. Review your submission information
3. Submit for certification
4. Wait for Amazon's review (typically 3-5 business days)

## Monitoring and Maintenance

### 1. Set Up Logging

#### CloudWatch Logs
```javascript
console.log('Request received:', JSON.stringify(handlerInput.requestEnvelope));
console.log('Claude API response time:', responseTime + 'ms');
```

### 2. Implement Error Tracking

#### Using CloudWatch Metrics
```javascript
// In your Lambda function
const AWS = require('aws-sdk');
const cloudWatch = new AWS.CloudWatch({apiVersion: '2010-08-01'});

// Track errors
async function trackError(errorType) {
  try {
    await cloudWatch.putMetricData({
      Namespace: 'ClaudeAlexaSkill',
      MetricData: [
        {
          MetricName: 'Errors',
          Dimensions: [
            {
              Name: 'ErrorType',
              Value: errorType
            }
          ],
          Value: 1,
          Unit: 'Count'
        }
      ]
    }).promise();
  } catch (error) {
    console.error('Failed to log metrics:', error);
  }
}
```

### 3. Regular Updates

1. Monitor for Alexa Skills Kit updates
2. Check for Claude API changes
3. Review and update system prompts
4. Add new features based on user feedback

## Advanced Features

### 1. Multi-Turn Conversations

Enhance your skill to handle complex multi-turn dialogues:

```javascript
// In your AskClaudeIntentHandler
if (needsMoreInfo) {
  return handlerInput.responseBuilder
    .speak(claudeResponse)
    .reprompt('Can you provide more details?')
    .addElicitSlotDirective('FollowUpInfo')
    .getResponse();
}
```

### 2. Alexa Presentation Language (APL) Support

If users have Echo Show or other screen devices:

```javascript
// Check if device supports display
if (Alexa.getSupportedInterfaces(handlerInput.requestEnvelope)['Alexa.Presentation.APL']) {
  // Add APL directive
  handlerInput.responseBuilder.addDirective({
    type: 'Alexa.Presentation.APL.RenderDocument',
    document: require('./apl-template.json'),
    datasources: {
      claudeData: {
        type: 'object',
        properties: {
          response: claudeResponse,
          title: 'Claude Says:',
        }
      }
    }
  });
}
```

### 3. Account Linking

To associate Anthropic accounts with Alexa users:

1. Go to the "Account Linking" tab in the Alexa Developer Console
2. Set up OAuth 2.0 authorization
3. Configure client ID, secret, and authorization URL
4. Define scope permissions
5. Set up redirect URLs

## Troubleshooting

### 1. Common Issues and Solutions

#### Response Too Long
```javascript
// Automatically truncate long responses
if (claudeResponse.length > 8000) {
  claudeResponse = claudeResponse.substring(0, 7500) + '... That\'s the summary. Would you like me to continue?';
}
```

#### API Timeout
```javascript
// Set timeout for API calls
const apiResponse = await Promise.race([
  anthropicClient.generateResponse(formattedQuery),
  new Promise((_, reject) => 
    setTimeout(() => reject(new Error('API timeout')), 5000)
  )
]).catch(error => {
  console.error('API call failed:', error);
  return 'I\'m having trouble connecting right now. Please try again in a moment.';
});
```

#### Session Expiration
```javascript
// Check if session has expired
if (handlerInput.requestEnvelope.session.new && sessionAttributes.lastQuery) {
  speakOutput = 'Welcome back! We were discussing ' + sessionAttributes.lastTopic + '. Would you like to continue?';
}
```

### 2. Debug Logging

```javascript
// Enhanced debug logging
function logDebug(message, data) {
  if (process.env.DEBUG_MODE === 'true') {
    console.log(`[DEBUG] ${message}`, JSON.stringify(data, null, 2));
  }
}
```

### package.json

```json
{
  "name": "claude-alexa-skill",
  "version": "1.0.0",
  "description": "Claude Assistant for Alexa",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [
    "alexa",
    "skill",
    "claude",
    "ai"
  ],
  "author": "Your Name",
  "license": "MIT",
  "dependencies": {
    "ask-sdk-core": "^2.12.1",
    "ask-sdk-model": "^1.38.1",
    "axios": "^0.27.2"
  }
}
```

### skill.json

```json
{
  "manifest": {
    "publishingInformation": {
      "locales": {
        "en-US": {
          "summary": "Chat with Claude, Anthropic's AI assistant, through your Alexa device.",
          "examplePhrases": [
            "Alexa, ask Claude Assistant what is quantum computing",
            "Alexa, ask Claude Assistant to explain climate change",
            "Alexa, ask Claude Assistant for a pasta recipe"
          ],
          "keywords": [
            "ai",
            "assistant",
            "claude",
            "anthropic",
            "chat",
            "help",
            "knowledge"
          ],
          "name": "Claude Assistant",
          "description": "Access the intelligence of Claude, Anthropic's AI assistant, through your Alexa device. Ask questions on a wide range of topics, get explanations of complex concepts, request creative content, or simply have a conversation. Claude can help with information, explanations, ideas, and more - all through the convenience of voice commands on your Alexa device.\n\nClaude is designed to be helpful, harmless, and honest, providing thoughtful and nuanced responses to your questions. The skill maintains context throughout your conversation, allowing for natural follow-up questions and deeper discussions.\n\nTo use the skill, simply say 'Alexa, ask Claude Assistant' followed by your question or request."
        }
      },
      "isAvailableWorldwide": true,
      "testingInstructions": "Launch the skill by saying 'Alexa, open Claude Assistant' or 'Alexa, ask Claude Assistant [question]'. The skill should maintain conversation context across multiple questions.",
      "category": "KNOWLEDGE_AND_TRIVIA",
      "distributionCountries": []
    },
    "apis": {
      "custom": {
        "endpoint": {
          "uri": "arn:aws:lambda:us-east-1:YOUR_AWS_ACCOUNT_ID:function:ClaudeAlexaSkill"
        },
        "interfaces": []
      }
    },
    "manifestVersion": "1.0",
    "permissions": [],
    "privacyAndCompliance": {
      "allowsPurchases": false,
      "isExportCompliant": true,
      "containsAds": false,
      "isChildDirected": false,
      "usesPersonalInfo": false
    }
  }
}
```

## Example Implementation

### Complete Lambda Function (Node.js)

```javascript
const Alexa = require('ask-sdk-core');
const axios = require('axios');

// Claude API client
const claudeClient = {
  async getResponse(prompt, conversationHistory = []) {
    try {
      const startTime = Date.now();
      
      const response = await axios.post(
        'https://api.anthropic.com/v1/messages',
        {
          model: process.env.MODEL_NAME || 'claude-3-7-sonnet-20250219',
          messages: [...conversationHistory, { role: 'user', content: prompt }],
          max_tokens: 1024
        },
        {
          headers: {
            'Content-Type': 'application/json',
            'x-api-key': process.env.ANTHROPIC_API_KEY,
            'anthropic-version': '2023-06-01'
          },
          timeout: 8000
        }
      );
      
      const elapsedTime = Date.now() - startTime;
      console.log(`Claude API response time: ${elapsedTime}ms`);
      
      return response.data.content[0].text;
    } catch (error) {
      console.error('Claude API error:', error.response ? error.response.data : error.message);
      return "I'm having trouble connecting to my systems right now. Please try again later.";
    }
  }
};

// Helper function to format responses for voice
function formatForVoice(text) {
  // Replace markdown with voice-friendly formatting
  text = text.replace(/\*\*(.*?)\*\*/g, '$1');
  text = text.replace(/\*(.*?)\*/g, '$1');
  text = text.replace(/```[\s\S]*?```/g, 'I would share code here, but it\'s better viewed than heard.');
  text = text.replace(/\[(.*?)\]\((.*?)\)/g, '$1');
  
  // Replace bullet points
  text = text.replace(/^- (.*)/gm, '<break time="300ms"/> $1');
  
  // Add pauses for better cadence
  text = text.replace(/\. /g, '.<break time="200ms"/> ');
  text = text.replace(/\! /g, '!<break time="300ms"/> ');
  text = text.replace(/\? /g, '?<break time="300ms"/> ');
  
  // Limit length
  if (text.length > 8000) {
    text = text.substring(0, 7500) + '<break time="500ms"/> I have more to share on this topic. Would you like me to continue?';
  }
  
  return `<speak>${text}</speak>`;
}

// LaunchRequest handler
const LaunchRequestHandler = {
  canHandle(handlerInput) {
    return Alexa.getRequestType(handlerInput.requestEnvelope) === 'LaunchRequest';
  },
  handle(handlerInput) {
    const speakOutput = 'Welcome to Claude Assistant. I can answer questions, provide information, or just chat. What would you like to know?';
    
    return handlerInput.responseBuilder
      .speak(speakOutput)
      .reprompt(speakOutput)
      .getResponse();
  }
};

// AskClaudeIntent handler
const AskClaudeIntentHandler = {
  canHandle(handlerInput) {
    return Alexa.getRequestType(handlerInput.requestEnvelope) === 'IntentRequest'
      && Alexa.getIntentName(handlerInput.requestEnvelope) === 'AskClaudeIntent';
  },
  async handle(handlerInput) {
    const query = Alexa.getSlotValue(handlerInput.requestEnvelope, 'Query');
    
    // Get session attributes for conversation history
    const sessionAttributes = handlerInput.attributesManager.getSessionAttributes();
    if (!sessionAttributes.conversationHistory) {
      sessionAttributes.conversationHistory = [];
    }
    
    // Track conversation topic
    sessionAttributes.lastQuery = query;
    
    // Determine if this is a new topic
    let topicChanged = true;
    if (sessionAttributes.conversationHistory.length > 0) {
      // Simple heuristic - check if query contains words from previous exchanges
      const previousExchanges = sessionAttributes.conversationHistory
        .filter(msg => msg.role === 'user')
        .map(msg => msg.content)
        .join(' ')
        .toLowerCase();
      
      const queryWords = query.toLowerCase().split(/\s+/);
      const commonWords = queryWords.filter(word => 
        word.length > 3 && previousExchanges.includes(word)
      );
      
      topicChanged = commonWords.length < 2;
    }
    
    // Reset conversation history if topic changed
    if (topicChanged && sessionAttributes.conversationHistory.length > 0) {
      sessionAttributes.conversationHistory = [];
      sessionAttributes.currentTopic = extractTopic(query);
    }
    
    // Set a system prompt for Claude
    const systemPrompt = `You are Claude, an AI assistant created by Anthropic, now available through an Alexa Skill. 
Keep your responses concise and suitable for voice. Limit responses to 3-4 sentences unless the user specifically asks for more detail.
Avoid mentioning URLs, visual elements, or formatting that doesn't work in voice interfaces.
The user is speaking to you through Alexa, so frame your responses accordingly.`;
    
    // Prepare the prompt with conversation context
    let formattedQuery = query;
    if (sessionAttributes.conversationHistory.length > 0) {
      formattedQuery = `This is a continuing conversation. ${query}`;
    }
    
    try {
      // Call Claude API
      let claudeResponse = await claudeClient.getResponse(formattedQuery, sessionAttributes.conversationHistory);
      
      // Format for voice
      const voiceResponse = formatForVoice(claudeResponse);
      
      // Update conversation history
      sessionAttributes.conversationHistory.push(
        { role: 'user', content: query },
        { role: 'assistant', content: claudeResponse }
      );
      
      // Keep only last 6 messages to manage context window
      if (sessionAttributes.conversationHistory.length > 6) {
        sessionAttributes.conversationHistory = sessionAttributes.conversationHistory.slice(-6);
      }
      
      handlerInput.attributesManager.setSessionAttributes(sessionAttributes);
      
      return handlerInput.responseBuilder
        .speak(voiceResponse)
        .reprompt('Is there anything else you would like to know?')
        .getResponse();
    } catch (error) {
      console.error('Error handling query:', error);
      return handlerInput.responseBuilder
        .speak('I encountered a problem processing your request. Please try again.')
        .reprompt('You can ask me another question or try rephrasing.')
        .getResponse();
    }
  }
};

// Helper function to extract topic from query
function extractTopic(query) {
  const words = query.toLowerCase().split(/\s+/);
  const stopWords = ['what', 'when', 'where', 'who', 'why', 'how', 'is', 'are', 'was', 'were', 'do', 'does', 'did', 'can', 'could', 'should', 'would', 'a', 'an', 'the', 'and', 'but', 'or', 'for', 'nor', 'on', 'at', 'to', 'from', 'by'];
  
  const contentWords = words.filter(word => !stopWords.includes(word) && word.length > 3);
  return contentWords.length > 0 ? contentWords[0] : 'general information';
}

// HelpIntent handler
const HelpIntentHandler = {
  canHandle(handlerInput) {
    return Alexa.getRequestType(handlerInput.requestEnvelope) === 'IntentRequest'
      && Alexa.getIntentName(handlerInput.requestEnvelope) === 'AMAZON.HelpIntent';
  },
  handle(handlerInput) {
    const speakOutput = 'You can ask me any question and I\'ll use Claude\'s intelligence to answer. For example, try saying "ask Claude what is quantum computing?" or "ask Claude to explain climate change" or even "ask Claude for a quick pasta recipe."';
    
    return handlerInput.responseBuilder
      .speak(speakOutput)
      .reprompt(speakOutput)
      .getResponse();
  }
};

// Cancel and Stop Intent handler
const CancelAndStopIntentHandler = {
  canHandle(handlerInput) {
    return Alexa.getRequestType(handlerInput.requestEnvelope) === 'IntentRequest'
      && (Alexa.getIntentName(handlerInput.requestEnvelope) === 'AMAZON.CancelIntent'
          || Alexa.getIntentName(handlerInput.requestEnvelope) === 'AMAZON.StopIntent');
  },
  handle(handlerInput) {
    const speakOutput = 'Goodbye! Feel free to ask Claude for help anytime.';
    
    return handlerInput.responseBuilder
      .speak(speakOutput)
      .getResponse();
  }
};

// FallbackIntent handler
const FallbackIntentHandler = {
  canHandle(handlerInput) {
    return Alexa.getRequestType(handlerInput.requestEnvelope) === 'IntentRequest'
      && Alexa.getIntentName(handlerInput.requestEnvelope) === 'AMAZON.FallbackIntent';
  },
  handle(handlerInput) {
    const speakOutput = 'I\'m not sure how to help with that. You can ask me a question by saying "ask Claude" followed by your question.';
    
    return handlerInput.responseBuilder
      .speak(speakOutput)
      .reprompt(speakOutput)
      .getResponse();
  }
};

// SessionEndedRequest handler
const SessionEndedRequestHandler = {
  canHandle(handlerInput) {
    return Alexa.getRequestType(handlerInput.requestEnvelope) === 'SessionEndedRequest';
  },
  handle(handlerInput) {
    console.log(`Session ended with reason: ${handlerInput.requestEnvelope.request.reason}`);
    return handlerInput.responseBuilder.getResponse();
  }
};

// Error handler
const ErrorHandler = {
  canHandle() {
    return true;
  },
  handle(handlerInput, error) {
    console.error(`Error handled: ${error.stack}`);
    
    const speakOutput = 'Sorry, I encountered a problem. Please try again later.';
    
    return handlerInput.responseBuilder
      .speak(speakOutput)
      .reprompt(speakOutput)
      .getResponse();
  }
};

// Export handler function
exports.handler = Alexa.SkillBuilders.custom()
  .addRequestHandlers(
    LaunchRequestHandler,
    AskClaudeIntentHandler,
    HelpIntentHandler,
    CancelAndStopIntentHandler,
    FallbackIntentHandler,
    SessionEndedRequestHandler
  )
  .addErrorHandlers(ErrorHandler)
  .lambda();
```

## Conclusion

Building a Claude Skill for Alexa involves several key components:

1. **Setting up your development environment** with the necessary accounts and tools
2. **Creating an Alexa skill** with appropriate invocation name and intents
3. **Developing a Lambda function** to handle requests and responses
4. **Integrating Claude's API** to generate intelligent responses
5. **Optimizing responses for voice** with proper formatting and SSML
6. **Managing conversation context** through session attributes
7. **Testing and debugging** across multiple platforms
8. **Deploying and certifying** your skill through Amazon

By following this guide, you've created a bridge between Alexa's voice interface and Claude's AI capabilities, offering users a seamless way to access Claude's intelligence through their Alexa devices.

Future enhancements could include:
- Adding authentication to limit access or customize responses
- Implementing user profiles to personalize interactions
- Expanding to multiple languages
- Adding visual responses for screen-enabled devices
- Incorporating other Amazon services like music, shopping, or smart home controls

Remember to regularly update your skill as both Alexa and Claude evolve, ensuring you're always offering the best possible experience to your users.

## Further Reading

Here are some valuable resources to help you develop and expand your Claude-Alexa integration:

### Amazon Alexa Documentation
- [Alexa Skills Kit Documentation](https://developer.amazon.com/en-US/docs/alexa/ask-overviews/what-is-the-alexa-skills-kit.html) - Comprehensive guide to building Alexa skills
- [ASK SDK for Node.js](https://developer.amazon.com/en-US/docs/alexa/alexa-skills-kit-sdk-for-nodejs/overview.html) - Official SDK documentation
- [Voice Design Guide](https://developer.amazon.com/en-US/docs/alexa/alexa-design/get-started.html) - Best practices for voice interfaces
- [SSML Reference](https://developer.amazon.com/en-US/docs/alexa/custom-skills/speech-synthesis-markup-language-ssml-reference.html) - Guide to Speech Synthesis Markup Language
- [Alexa Skills Testing Guide](https://developer.amazon.com/en-US/docs/alexa/devconsole/test-your-skill.html) - Methods for testing Alexa skills

### Claude API Documentation
- [Anthropic API Documentation](https://docs.anthropic.com/en/docs/) - Official Claude API reference
- [Claude API Cookbook](https://docs.anthropic.com/en/docs/cookbook) - Code examples and patterns
- [Claude Prompt Design Guide](https://docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/overview) - Best practices for prompting Claude

### AWS Lambda Resources
- [AWS Lambda Developer Guide](https://docs.aws.amazon.com/lambda/latest/dg/welcome.html) - Comprehensive Lambda documentation
- [AWS Lambda Node.js Guide](https://docs.aws.amazon.com/lambda/latest/dg/lambda-nodejs.html) - Node.js specific Lambda information
- [CloudWatch Logging for Lambda](https://docs.aws.amazon.com/lambda/latest/dg/monitoring-cloudwatchlogs.html) - Guide to monitoring Lambda functions

### Community Resources
- [Alexa Skills Kit GitHub](https://github.com/alexa/alexa-skills-kit-sdk-for-nodejs) - Open source SDK and examples
- [Alexa Developer Forums](https://developer.amazon.com/en-US/blogs/alexa/alexa-skills-kit) - Community discussion and support
- [Anthropic Discord Community](https://discord.gg/anthropic) - Connect with other Claude developers

### Tools and Utilities
- [Bespoken Tools](https://bespoken.io/alexa-skills-testing/) - Tools for testing voice applications
- [Postman](https://www.postman.com/) - API development and testing tool
- [Alexa Skills Kit CLI](https://developer.amazon.com/en-US/docs/alexa/smapi/quick-start-alexa-skills-kit-command-line-interface.html) - Command line tools for skill development