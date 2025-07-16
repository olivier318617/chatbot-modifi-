<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Chatbot N8N</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            margin: 0;
            padding: 20px;
            background-color: #f0f0f0;
        }
        
        .chat-container {
            max-width: 600px;
            margin: 0 auto;
            background-color: white;
            border-radius: 10px;
            box-shadow: 0 2px 10px rgba(0,0,0,0.1);
            overflow: hidden;
        }
        
        .chat-header {
            background-color: #4a5568;
            color: white;
            padding: 15px;
            text-align: center;
        }
        
        .chat-messages {
            height: 400px;
            overflow-y: auto;
            padding: 20px;
            background-color: #f7fafc;
        }
        
        .message {
            margin-bottom: 15px;
            padding: 10px 15px;
            border-radius: 10px;
            max-width: 70%;
        }
        
        .user-message {
            background-color: #4299e1;
            color: white;
            margin-left: auto;
            text-align: right;
        }
        
        .bot-message {
            background-color: #e2e8f0;
            color: #2d3748;
        }
        
        .chat-input {
            display: flex;
            padding: 15px;
            background-color: white;
            border-top: 1px solid #e2e8f0;
        }
        
        #messageInput {
            flex: 1;
            padding: 10px;
            border: 1px solid #cbd5e0;
            border-radius: 5px;
            font-size: 16px;
        }
        
        #sendButton {
            margin-left: 10px;
            padding: 10px 20px;
            background-color: #4299e1;
            color: white;
            border: none;
            border-radius: 5px;
            cursor: pointer;
            font-size: 16px;
        }
        
        #sendButton:hover {
            background-color: #3182ce;
        }
        
        #sendButton:disabled {
            background-color: #cbd5e0;
            cursor: not-allowed;
        }
        
        .typing-indicator {
            display: none;
            padding: 10px;
            font-style: italic;
            color: #718096;
        }
    </style>
</head>
<body>
    <div class="chat-container">
        <div class="chat-header">
            <h2>Assistant N8N</h2>
        </div>
        
        <div class="chat-messages" id="chatMessages">
            <div class="message bot-message">
                Bonjour ! Comment puis-je vous aider aujourd'hui ?
            </div>
        </div>
        
        <div class="typing-indicator" id="typingIndicator">
            L'assistant est en train d'écrire...
        </div>
        
        <div class="chat-input">
            <input 
                type="text" 
                id="messageInput" 
                placeholder="Tapez votre message..."
                onkeypress="handleKeyPress(event)"
            />
            <button id="sendButton" onclick="sendMessage()">Envoyer</button>
        </div>
    </div>

    <script>
        const webhookUrl = "https://n8n.srv773091.hstgr.cloud/webhook/74e97935-e9cb-44da-9726-b20aafc43102/chat";
        
        function handleKeyPress(event) {
            if (event.key === 'Enter') {
                sendMessage();
            }
        }
        
        async function sendMessage() {
            const input = document.getElementById('messageInput');
            const message = input.value.trim();
            
            if (!message) return;
            
            // Désactiver l'input pendant l'envoi
            input.disabled = true;
            document.getElementById('sendButton').disabled = true;
            
            // Ajouter le message de l'utilisateur
            addMessage(message, 'user');
            
            // Vider l'input
            input.value = '';
            
            // Afficher l'indicateur de frappe
            document.getElementById('typingIndicator').style.display = 'block';
            
            try {
                // Envoyer la requête au webhook N8N
                const response = await fetch(webhookUrl, {
                    method: 'POST',
                    headers: {
                        'Content-Type': 'application/json',
                    },
                    body: JSON.stringify({
                        message: message,
                        timestamp: new Date().toISOString()
                    })
                });
                
                if (!response.ok) {
                    throw new Error('Erreur de connexion');
                }
                
                // Debug : afficher la réponse brute
                const responseText = await response.text();
                console.log('Réponse brute du webhook:', responseText);
                
                let data;
                try {
                    data = JSON.parse(responseText);
                } catch (e) {
                    // Si ce n'est pas du JSON, utiliser directement le texte
                    console.log('Réponse non-JSON reçue');
                    addMessage(responseText, 'bot');
                    return;
                }
                
                // Debug : afficher la structure de données
                console.log('Structure de données reçue:', data);
                
                // Essayer différentes structures possibles
                let botResponse;
                
                // Si la réponse est directement une chaîne
                if (typeof data === 'string') {
                    botResponse = data;
                }
                // Si c'est un objet, chercher dans différents champs possibles
                else if (typeof data === 'object') {
                    botResponse = data.message || 
                                 data.response || 
                                 data.text || 
                                 data.output ||
                                 data.answer ||
                                 data.reply ||
                                 data.content ||
                                 data.body ||
                                 JSON.stringify(data); // Afficher l'objet complet si aucun champ connu
                }
                
                addMessage(botResponse, 'bot');
                
            } catch (error) {
                console.error('Erreur:', error);
                addMessage('Désolé, une erreur s\'est produite. Veuillez réessayer.', 'bot');
            } finally {
                // Masquer l'indicateur de frappe
                document.getElementById('typingIndicator').style.display = 'none';
                
                // Réactiver l'input
                input.disabled = false;
                document.getElementById('sendButton').disabled = false;
                input.focus();
            }
        }
        
        function addMessage(text, sender) {
            const messagesContainer = document.getElementById('chatMessages');
            const messageDiv = document.createElement('div');
            messageDiv.className = `message ${sender}-message`;
            messageDiv.textContent = text;
            messagesContainer.appendChild(messageDiv);
            
            // Faire défiler vers le bas
            messagesContainer.scrollTop = messagesContainer.scrollHeight;
        }
    </script>
</body>
</html>
