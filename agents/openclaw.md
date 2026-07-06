{
  "mcp": {
    "servers": {
      "playwright": {
        "command": "node",
        "args": [
          "/mnt/openclaw/.openclaw/mcp-bin/node_modules/@playwright/mcp/cli.js",
          "--headless",
          "--executable-path",
          "/opt/playwright/chromium-1217/chrome-linux64/chrome"
        ]
      }
    }
  },
  "meta": {
    "lastTouchedVersion": "2026.3.8",
    "lastTouchedAt": "2026-04-25T08:46:31.749Z"
  },
  "wizard": {
    "lastRunAt": "2026-02-25T07:35:51.386Z",
    "lastRunVersion": "2026.2.25",
    "lastRunCommand": "onboard",
    "lastRunMode": "local"
  },
  "update": {
    "checkOnStart": false
  },
  "browser": {
    "attachOnly": true,
    "defaultProfile": "openclaw",
    "profiles": {
      "openclaw": {
        "cdpUrl": "http://127.0.0.1:9222",
        "color": "#00FF00"
      }
    }
  },
  "models": {
    "mode": "replace",
    "providers": {
      "kubeplex-maas": {
        "baseUrl": "https://mmc.sankuai.com/openclaw/v1",
        "apiKey": "catpaw",
        "api": "openai-completions",
        "headers": {
          "X-User-Id": "yanglin71",
          "X-Conversation-Id": "uq3pya1m4sqaqay9qgtr",
          "X-Scene": "CATCLAW"
        },
        "models": [
          {
            "id": "catclaw-proxy-model",
            "name": "CATCLAW_PROXY_MODEL",
            "input": [
              "text",
              "image"
            ],
            "contextWindow": 150000
          }
        ]
      },
      "kubeplex-maas-saferoom": {
        "baseUrl": "https://mmc.sankuai.com/openclaw/v1",
        "apiKey": "catpaw",
        "api": "openai-completions",
        "headers": {
          "X-User-Id": "yanglin71",
          "X-Conversation-Id": "uq3pya1m4sqaqay9qgtr",
          "X-Scene": "CATCLAW",
          "X-SafeRoom-Token": "saferoom"
        },
        "models": [
          {
            "id": "catclaw-proxy-model",
            "name": "CATCLAW_PROXY_MODEL",
            "input": [
              "text",
              "image"
            ],
            "contextWindow": 150000
          }
        ]
      }
    }
  },
  "agents": {
    "defaults": {
      "model": {
        "primary": "kubeplex-maas/catclaw-proxy-model"
      },
      "memorySearch": {
        "enabled": true,
        "sources": [
          "memory",
          "sessions"
        ],
        "experimental": {
          "sessionMemory": true
        },
        "provider": "openai",
        "remote": {
          "baseUrl": "https://mmc.sankuai.com/openclaw/v1/native",
          "apiKey": "catpaw",
          "headers": {
            "X-User-Id": "yanglin71",
            "X-Conversation-Id": "uq3pya1m4sqaqay9qgtr",
            "X-Scene": "CATCLAW"
          }
        },
        "fallback": "none",
        "model": "text-embedding-3-large",
        "sync": {
          "intervalMinutes": 1,
          "sessions": {
            "deltaBytes": 100000,
            "deltaMessages": 50
          }
        },
        "cache": {
          "enabled": true,
          "maxEntries": 50000
        }
      },
      "compaction": {
        "mode": "default",
        "keepRecentTokens": 10000,
        "reserveTokensFloor": 20000,
        "memoryFlush": {
          "enabled": true,
          "softThresholdTokens": 4000,
          "prompt": "Write any lasting notes to memory/YYYY-MM-DD.md; reply with NO_REPLY if nothing to store.",
          "systemPrompt": "Session nearing compaction. Store durable memories now."
        }
      },
      "heartbeat": {
        "every": "48h"
      },
      "maxConcurrent": 4,
      "subagents": {
        "maxConcurrent": 8
      },
      "timeoutSeconds": 1800,
      "llm": {
        "idleTimeoutSeconds": 0
      },
      "imageMaxDimensionPx": 2000
    },
    "list": [
      {
        "id": "main",
        "subagents": {
          "allowAgents": [
            "pm",
            "frontend",
            "backend",
            "qa"
          ]
        }
      },
      {
        "id": "saferoom",
        "name": "safeRoom",
        "workspace": "/root/.openclaw/workspace-saferoom",
        "agentDir": "/root/.openclaw/agents/saferoom/agent",
        "model": {
          "primary": "kubeplex-maas-saferoom/catclaw-proxy-model"
        }
      },
      {
        "id": "pm",
        "name": "王磊",
        "workspace": "/mnt/openclaw/.openclaw/workspace-pm",
        "agentDir": "/root/.openclaw/agents/pm/agent",
        "model": {
          "primary": "kubeplex-maas/catclaw-proxy-model"
        }
      },
      {
        "id": "frontend",
        "name": "张晨",
        "workspace": "/mnt/openclaw/.openclaw/workspace-frontend",
        "agentDir": "/root/.openclaw/agents/frontend/agent",
        "model": {
          "primary": "kubeplex-maas/catclaw-proxy-model"
        }
      },
      {
        "id": "backend",
        "name": "陈浩",
        "workspace": "/mnt/openclaw/.openclaw/workspace-backend",
        "agentDir": "/root/.openclaw/agents/backend/agent",
        "model": {
          "primary": "kubeplex-maas/catclaw-proxy-model"
        }
      },
      {
        "id": "qa",
        "name": "张伟",
        "workspace": "/mnt/openclaw/.openclaw/workspace-qa",
        "agentDir": "/root/.openclaw/agents/qa/agent",
        "model": {
          "primary": "kubeplex-maas/catclaw-proxy-model"
        }
      }
    ]
  },
  "tools": {
    "deny": [
      "web_search"
    ],
    "sessions": {
      "visibility": "all"
    },
    "agentToAgent": {
      "enabled": true
    },
    "web": {
      "fetch": {
        "maxChars": 30000,
        "maxCharsCap": 30000
      }
    },
    "exec": {
      "notifyOnExit": false,
      "notifyOnExitEmptySuccess": false
    },
    "loopDetection": {
      "enabled": true,
      "globalCircuitBreakerThreshold": 30
    }
  },
  "messages": {
    "ackReactionScope": "group-mentions",
    "queue": {
      "mode": "interrupt"
    }
  },
  "commands": {
    "native": "auto",
    "nativeSkills": "auto",
    "restart": true,
    "ownerDisplay": "raw",
    "ownerAllowFrom": [
      "*"
    ]
  },
  "session": {
    "dmScope": "per-channel-peer",
    "maintenance": {
      "mode": "enforce",
      "maxDiskBytes": "50mb"
    }
  },
  "channels": {
    "daxiang": {
      "enabled": true,
      "bridgeUrl": "wss://openclaw-bridge.sankuai.com/ws",
      "bridgeSecret": "",
      "clientId": "9191f25b19",
      "clientSecret": "58fb2eb3db794128b2e120c8cd56e94a",
      "botId": "139058069533",
      "botMisId": "uq3pya1m4sqaqay9qgtr",
      "dmPolicy": "open",
      "allowFrom": [
        "*"
      ],
      "groupPolicy": "open",
      "groupAllowFrom": [
        "*"
      ],
      "requireMention": false,
      "historyLimit": 10,
      "textChunkLimit": 4000,
      "dmHistoryLimit": 30,
      "safeRoomAgentId": "saferoom"
    }
  },
  "gateway": {
    "port": 18789,
    "mode": "local",
    "bind": "lan",
    "trustedProxies": [
      "10.0.0.0/8",
      "172.16.0.0/12",
      "192.168.0.0/16"
    ],
    "controlUi": {
      "allowedOrigins": [
        "*"
      ],
      "dangerouslyAllowHostHeaderOriginFallback": true,
      "dangerouslyDisableDeviceAuth": true
    },
    "auth": {
      "mode": "token",
      "token": "catpaw"
    },
    "tailscale": {
      "mode": "off",
      "resetOnExit": false
    }
  },
  "skills": {
    "load": {
      "extraDirs": [
        "/app/skills"
      ],
      "watch": true,
      "watchDebounceMs": 3000
    },
    "install": {
      "nodeManager": "pnpm"
    },
    "limits": {
      "maxSkillsPromptChars": 30000
    }
  },
  "plugins": {
    "slots": {
      "memory": "catpaw-memory"
    },
    "entries": {
      "daxiang": {
        "enabled": true
      },
      "message-logger-plugin": {
        "enabled": true,
        "config": {
          "agentStuckAlert": {
            "webhookUrl": "https://yuntu.sankuai.com/webhook/catclaw/agent-stuck"
          }
        },
        "hooks": {
          "allowConversationAccess": true
        }
      },
      "catpaw-memory": {
        "enabled": true,
        "hooks": {
          "allowConversationAccess": true
        }
      },
      "memory-core": {
        "enabled": false
      },
      "memory-lancedb": {
        "enabled": false
      },
      "catclawsec": {
        "enabled": true,
        "hooks": {
          "allowConversationAccess": true
        }
      }
    },
    "allow": [
      "catclawsec",
      "catpaw-memory",
      "daxiang",
      "message-logger-plugin"
    ],
    "load": {
      "paths": [
        "/app/defaults/openclaw/.openclaw/extensions"
      ]
    }
  },
  "cron": {
    "sessionRetention": "1h"
  }
}
