🌱 Agrishe — Smart Irrigation Workflow (n8n + Groq + OpenWeather)
🧠 Overview

Agrishe is an intelligent irrigation management workflow built with n8n, designed to make agricultural automation a little less dumb.
It combines real-time soil humidity data, weather forecasts, and a sprinkle of AI reasoning to decide whether irrigation is necessary.

The workflow provides:

A quick yes/no irrigation decision.

A concise, explainable recommendation.

A Tunisian dialect agriculture assistant for local user interaction.

Optional MongoDB integration for data persistence (e.g. orders, logs).

🧩 Workflow Architecture
Main Components
Node	Purpose
Webhook	Receives incoming POST requests with soil humidity data (humidity_values array).
AI Agent (LangChain)	Analyzes soil humidity and weather data to decide on irrigation.
Groq Chat Model	Provides the LLM reasoning (using deepseek-r1-distill-llama-70b via Groq).
OpenWeatherMap	Fetches weather forecast for the configured city (default: Sfax).
Respond to Webhook	Sends the irrigation recommendation back to the requester.
Webhook1	Handles general user queries (in Tunisian dialect).
AI Agent1	Answers user questions using contextual data and MongoDB access.
MongoDB	Provides data source for the Tunisian assistant (e.g. user records, orders).
Respond to Webhook1	Sends responses from the Tunisian assistant back to the user.
⚙️ Data Flow

Soil Sensor → Webhook

Sensor system sends humidity readings to the first webhook endpoint:

POST /webhook/21f7ea0d-be62-43b2-9fe5-e575f6e047f8
{
  "humidity_values": [1200, 1450, 2000, 1800]
}


AI Agent Decision

Agent uses the following rules:

If average humidity < 1500–2000, consider irrigation.

If rain expected in next 2 days, skip irrigation unless critically dry.

Produces two outputs:

Short answer: "yes" or "no"

Full message: "Irrigate — soil humidity is low and no rain expected."

Response

Workflow returns the decision instantly to the sender.

Tunisian Assistant Flow

For user interaction, another webhook (/webhook/ff57dff2-d500-4831-81a7-790bcbba5579) accepts queries in JSON:

{ "message": "chnou rayek fi arrosya tawa?" }


Response is generated in Tunisian Arabic, grounded on MongoDB data.

🌤️ Example Output

Request

{
  "humidity_values": [1300, 1500, 1400, 1600]
}


Response

{
  "short": "yes",
  "message": "Irrigate — soil humidity is low and no rain is expected in the next two days."
}

🔧 Setup Instructions
Prerequisites

n8n (>= 1.30)

Groq API key

OpenWeatherMap API key

MongoDB connection URI

Configuration

Import this workflow JSON into n8n.

Go to Credentials and add:

Groq account

OpenWeatherMap account

MongoDB account

Update the following parameters if needed:

City name in the OpenWeatherMap node.

Webhook paths (if redeployed on a different endpoint).

Activate the workflow.

🧪 Testing

You can test the irrigation logic with:

curl -X POST http://localhost:5678/webhook-test/21f7ea0d-be62-43b2-9fe5-e575f6e047f8 \
-H "Content-Type: application/json" \
-d '{"humidity_values":[1100, 1300, 1500]}'


Expected response:

{
  "short": "yes",
  "message": "Irrigate — soil is dry and no significant rainfall is expected."
}

🗣️ Tunisian Agriculture Assistant

Endpoint:

POST /webhook-test/ff57dff2-d500-4831-81a7-790bcbba5579


Example:

{
  "message": "chnou naamel ki el tmatem wallew yebs?"
}


Response (in Tunisian Arabic):

{
  "reply": "rawd t9is el s9i shwaya, tmatem ma7abhouch lma barcha tawa fil bardo."
}

💡 Notes

Humidity range assumed: 0 (dry) → 4095 (wet)

You can tweak the dryness threshold in the system message prompt.

For production, secure your webhooks with authentication or API keys.

Works seamlessly with ESP32-based soil sensor nodes.

🪴 Author

Eya — Engineering student, automating irrigation before the planet fully dries up.
Smart irrigation, but with style.
