# Comment-Bot
Personalized comment bot
import json
import re

# CHANGE THIS: Path to your exported chat
input_file = "chat.txt"  # or whatsapp_chat.txt
output_file = "training_data.jsonl"

conversations = []
current_conversation = []

with open(input_file, 'r', encoding='utf-8') as f:
    lines = f.readlines()

# Pattern for WhatsApp format: "12/04/23, 9:30 PM - You: message"
pattern = r'\d{2}/\d{2}/\d{2},.*?- (.*?): (.*)'

for line in lines:
    match = re.match(pattern, line)
    if match:
        speaker = match.group(1).strip()
        message = match.group(2).strip()
        
        # Skip empty messages
        if not message or message == "<Media omitted>":
            continue
            
        # Convert "You" to "user", others to "assistant" (your voice)
        role = "user" if "You" in speaker else "assistant"
        current_conversation.append({"role": role, "content": message})
        
        # Save every 10 exchanges as one training example
        if len(current_conversation) >= 20:  # 10 back-and-forths
            conversations.append({"messages": current_conversation})
            current_conversation = []

# Save last partial conversation
if current_conversation:
    conversations.append({"messages": current_conversation})

# Write to JSONL format (one JSON per line)
with open(output_file, 'w', encoding='utf-8') as f:
    for conv in conversations:
        f.write(json.dumps(conv) + '\n')

print(f"✅ Created {output_file} with {len(conversations)} conversations")
print(f"📊 Total messages: {sum(len(c['messages']) for c in conversations)}")
