<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Puter AI Chat</title>
    <style>
        :root {
            --bg-gradient: linear-gradient(135deg, #f5f7fb 0%, #e4e8f0 100%);
            --chat-bg: #ffffff;
            --primary: #4f46e5;
            --primary-hover: #4338ca;
            --text-main: #1f2937;
            --text-muted: #6b7280;
            --border: #e5e7eb;
            --user-bubble: #4f46e5;
            --ai-bubble: #f3f4f6;
        }

        body {
            font-family: 'Segoe UI', system-ui, -apple-system, sans-serif;
            background: var(--bg-gradient);
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            margin: 0;
            color: var(--text-main);
        }

        /* 💡 Main Card */
        .chat-container {
            background-color: var(--chat-bg);
            border-radius: 16px;
            box-shadow: 0 10px 25px -5px rgba(0, 0, 0, 0.05), 0 8px 10px -6px rgba(0, 0, 0, 0.05);
            width: 100%;
            max-width: 480px;
            height: 600px;
            display: flex;
            flex-direction: column;
            overflow: hidden;
            border: 1px solid var(--border);
        }

        /* 💡 Header */
        .chat-header {
            padding: 20px;
            border-bottom: 1px solid var(--border);
            background-color: #ffffff;
            display: flex;
            justify-content: space-between;
            align-items: center;
        }

        .header-text h2 {
            margin: 0;
            font-size: 1.1rem;
            font-weight: 600;
            color: var(--text-main);
        }

        .header-text p {
            margin: 4px 0 0 0;
            font-size: 0.8rem;
            color: var(--text-muted);
        }

        .clear-btn {
            background: none;
            border: 1px solid var(--border);
            color: var(--text-muted);
            padding: 6px 12px;
            font-size: 0.8rem;
            border-radius: 6px;
            cursor: pointer;
            transition: all 0.2s;
        }

        .clear-btn:hover {
            background-color: #f9fafb;
            color: #ef4444;
            border-color: #fecaca;
        }

        /* 💡 Chat Body & Message Bubbles */
        #response-output {
            flex: 1;
            padding: 20px;
            overflow-y: auto;
            background-color: #fafafa;
            display: flex;
            flex-direction: column;
            gap: 12px;
        }

        .message {
            max-width: 80%;
            padding: 10px 14px;
            border-radius: 14px;
            font-size: 0.95rem;
            line-height: 1.45;
            word-wrap: break-word;
            animation: slideUp 0.2s ease-out forwards;
        }

        .user-message {
            align-self: flex-end;
            background-color: var(--user-bubble);
            color: white;
            border-bottom-right-radius: 4px;
        }

        .ai-message {
            align-self: flex-start;
            background-color: var(--ai-bubble);
            color: var(--text-main);
            border-bottom-left-radius: 4px;
            border: 1px solid #e9eaeb;
        }

        .system-message {
            align-self: center;
            font-size: 0.8rem;
            color: var(--text-muted);
            background-color: transparent;
        }

        /* 💡 Input Area */
        .input-group {
            display: flex;
            gap: 10px;
            padding: 16px;
            background-color: #ffffff;
            border-top: 1px solid var(--border);
        }

        input[type="text"] {
            flex: 1;
            padding: 12px 16px;
            border: 1px solid var(--border);
            border-radius: 10px;
            font-size: 0.95rem;
            outline: none;
            transition: all 0.2s ease;
            background-color: #f9fafb;
        }

        input[type="text"]:focus {
            border-color: var(--primary);
            background-color: #ffffff;
            box-shadow: 0 0 0 3px rgba(79, 70, 229, 0.1);
        }

        .send-btn {
            padding: 0 20px;
            background-color: var(--primary);
            color: white;
            border: none;
            border-radius: 10px;
            font-size: 0.95rem;
            font-weight: 500;
            cursor: pointer;
            transition: all 0.2s ease;
            display: flex;
            align-items: center;
            justify-content: center;
        }

        .send-btn:hover {
            background-color: var(--primary-hover);
            box-shadow: 0 4px 6px -1px rgba(79, 70, 229, 0.2);
        }

        .send-btn:disabled {
            background-color: #a5b4fc;
            cursor: not-allowed;
            box-shadow: none;
        }

        /* Animations */
        @keyframes slideUp {
            from { opacity: 0; transform: translateY(10px); }
            to { opacity: 1; transform: translateY(0); }
        }

        /* Scrollbar */
        #response-output::-webkit-scrollbar { width: 6px; }
        #response-output::-webkit-scrollbar-thumb { background: #e5e7eb; border-radius: 3px; }
    </style>
</head>
<body>

    <div class="chat-container">
        <div class="chat-header">
            <div class="header-text">
                <h2>Puter AI Assistant</h2>
                <p>Powered by GPT-5.4 Nano</p>
            </div>
            <button class="clear-btn" id="clear-btn">Clear Chat</button>
        </div>
        
        <div id="response-output">
            <div class="message ai-message">Hello! I can now remember our conversation. How can I help you today?</div>
        </div>

        <div class="input-group">
            <input type="text" id="prompt-input" placeholder="Message..." aria-label="Prompt">
            <button class="send-btn" id="submit-btn">Send</button>
        </div>
    </div>

    <!-- Puter.js SDK -->
    <script src="https://js.puter.com/v2/"></script>
    
    <script>
        const inputField = document.getElementById('prompt-input');
        const submitBtn = document.getElementById('submit-btn');
        const clearBtn = document.getElementById('clear-btn');
        const outputDiv = document.getElementById('response-output');

        // 🧠 Array to store conversation history
        let messageHistory = [];

        // Helper to append messages to the screen
        function appendMessage(text, type) {
            const msgDiv = document.createElement('div');
            msgDiv.classList.add('message', `${type}-message`);
            msgDiv.innerText = text;
            outputDiv.appendChild(msgDiv);
            outputDiv.scrollTop = outputDiv.scrollHeight;
            return msgDiv;
        }

        function queryAI() {
            const promptValue = inputField.value.trim();
            if (!promptValue) return;

            // 1. Add User message to screen & history
            appendMessage(promptValue, 'user');
            messageHistory.push({ role: 'user', content: promptValue });
            
            inputField.value = '';

            // 2. Lock UI
            submitBtn.disabled = true;
            inputField.disabled = true;

            // 3. Add loading state
            const loadingBubble = appendMessage('Typing...', 'ai');
            loadingBubble.style.color = '#9ca3af';

            // 4. Puter AI Chat Call (Passing the entire message history)
            puter.ai.chat(messageHistory, {
                model: 'gpt-5.4-nano',
            }).then((response) => {
                loadingBubble.remove();
                
                const text = response.content || response;
                
                // Add AI message to screen & history
                appendMessage(text, 'ai');
                messageHistory.push({ role: 'assistant', content: text });
                
            }).catch((error) => {
                loadingBubble.remove();
                appendMessage(`Error: ${error.message}`, 'system');
                // Remove the last failed user message from history so it doesn't break future calls
                messageHistory.pop();
            }).finally(() => {
                // Unlock UI
                submitBtn.disabled = false;
                inputField.disabled = false;
                inputField.focus();
            });
        }

        // Feature: Clear Chat
        clearBtn.addEventListener('click', () => {
            messageHistory = [];
            outputDiv.innerHTML = `<div class="message ai-message">Chat cleared. Hello! How can I help you today?</div>`;
        });

        submitBtn.addEventListener('click', queryAI);

        inputField.addEventListener('keypress', function(e) {
            if (e.key === 'Enter') {
                queryAI();
            }
        });
    </script>
</body>
</html>
