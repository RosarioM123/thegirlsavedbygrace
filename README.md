from flask import Flask, request, jsonify
import psycopg2
import os
import openai

app = Flask(__name__)

# --- Database connection (placeholder env vars) ---
conn = psycopg2.connect(
    host=os.getenv("DB_HOST"),
    database=os.getenv("DB_NAME"),
    user=os.getenv("DB_USER"),
    password=os.getenv("DB_PASS")
)
cursor = conn.cursor()

# --- OpenAI setup ---
openai.api_key = os.getenv("OPENAI_API_KEY")

@app.route("/recommend", methods=["POST"])
def recommend_content():
    data = request.json
    user_text = data.get("text", "")

    # Query OpenAI for NLP-driven recommendation
    response = openai.ChatCompletion.create(
        model="gpt-4o-mini",
        messages=[
            {"role": "system", "content": "You are a mental health content recommender."},
            {"role": "user", "content": user_text}
        ]
    )

    recommendation = response["choices"][0]["message"]["content"]

    return jsonify({"recommendation": recommendation})


@app.route("/messages", methods=["POST"])
def save_message():
    data = request.json
    user_id = data["user_id"]
    message = data["message"]

    cursor.execute(
        "INSERT INTO messages (user_id, message) VALUES (%s, %s)",
        (user_id, message)
    )
    conn.commit()

    return jsonify({"status": "saved"})


@app.route("/messages/<int:user_id>", methods=["GET"])
def get_messages(user_id):
    cursor.execute("SELECT message FROM messages WHERE user_id = %s", (user_id,))
    rows = cursor.fetchall()
    return jsonify({"messages": [r[0] for r in rows]})


if __name__ == "__main__":
    app.run(debug=True)
