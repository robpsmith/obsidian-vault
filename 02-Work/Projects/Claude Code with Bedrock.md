Prerequisites                                                                                                   
  
  - AWS account with Bedrock access enabled                                                                       
  - Access to Claude models in Bedrock                                                                            
  - Appropriate IAM permissions                                                                                   
  
  Minimal Setup                                                                                                   
  
  # 1. Configure AWS credentials (pick one method)                                                                
  aws configure                          # Access keys  
  aws sso login --profile=my-profile     # SSO

  # 2. Set environment variables  
  export CLAUDE_CODE_USE_BEDROCK=1  
  export AWS_REGION=us-east-1            # Required — Claude Code doesn't read .aws config  
  export AWS_PROFILE=my-profile          # If using SSO

  # 3. Start Claude Code  
  claude

  Authentication Options  
  ┌─────────────────┬────────────────────────────────────────────────────────────────────────┐  
  │     Method      │                                 Setup                                  │  
  ├─────────────────┼────────────────────────────────────────────────────────────────────────┤  
  │ Access Keys     │ AWS_ACCESS_KEY_ID + AWS_SECRET_ACCESS_KEY + optional AWS_SESSION_TOKEN │  
  ├─────────────────┼────────────────────────────────────────────────────────────────────────┤  
  │ SSO Profile     │ aws sso login --profile=name then export AWS_PROFILE=name              │  
  ├─────────────────┼────────────────────────────────────────────────────────────────────────┤  
  │ Bedrock API Key │ export AWS_BEARER_TOKEN_BEDROCK=your-key                               │  
  └─────────────────┴────────────────────────────────────────────────────────────────────────┘  
  Customizing Models

  Default models are Claude Sonnet 4.5 (primary) and Claude Haiku 4.5 (fast). Override with:

  export ANTHROPIC_MODEL='global.anthropic.claude-sonnet-4-5-20250929-v1:0'  
  export ANTHROPIC_SMALL_FAST_MODEL='us.anthropic.claude-haiku-4-5-20251001-v1:0'

  IAM Permissions Needed

  At minimum, your IAM role/user needs:  
  - bedrock:InvokeModel  
  - bedrock:InvokeModelWithResponseStream  
  - bedrock:ListInferenceProfiles

  Auto-Refreshing Credentials (SSO)

  Add to ~/.claude/settings.json:

  {  
    "awsAuthRefresh": "aws sso login --profile myprofile",  
    "env": {  
      "AWS_PROFILE": "myprofile"  
    }  
  }

  Notes

  - /login and /logout commands are disabled when using Bedrock  
  - If you get "on-demand throughput isn't supported" errors, use an inference profile ID instead of a model ARN  
  - Prompt caching may not be available in all regions — set DISABLE_PROMPT_CACHING=1 if needed




In AWS, create a bedrock key:  
IAM > User > Security Credentials > Create bedrock api

In Control panel:  
System > edit environment variables > New  
Variable name: PATH  
Variable value > C:\Users\YourUser\.local\.bin

In cmd:  
Install git for bash at [https://git-scm.com/install/windows](https://git-scm.com/install/windows "https://git-scm.com/install/windows")  
irm [https://claude.ai/install.ps1](https://claude.ai/install.ps1 "https://claude.ai/install.ps1") | iex  
# You may have to restart cmd or open a new tab at this point  
$env:CLAUDE_CODE_USE_BEDROCK="1"  
$env:AWS_REGION="us-east-1"  
$env:AWS_BEARER_TOKEN_BEDROCK="Bedrock API key"