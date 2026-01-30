# virtual-therapist

> A conversational AI chatbot that provides psychology-focused FAQ responses based on therapist forum discussions

Built with [Rasa](https://rasa.com), this retrieval-based chatbot matches user mental health queries to curated therapist responses, making preliminary psychological guidance accessible through a simple web interface.

## Usage

Ask the chatbot about mental health topics and get responses from trained therapists:

```bash
# Start the Rasa server
rasa run --cors '*' --port 5005

# Open index.html directly in your web browser, or serve via HTTP:
# python -m http.server 8000
# Then navigate to: http://localhost:8000/index.html
```

**Example conversation:**

```
User: "My husband cheated on me and I'm struggling to forgive him"
Bot:  [Returns therapist advice on handling infidelity and rebuilding trust]

User: "I'm feeling anxious about starting therapy"
Bot:  [Provides guidance on what to expect in therapy sessions]
```

The chatbot uses natural language understanding to match your question to the most relevant response from its FAQ database of therapist forum discussions.

## Installation

```bash
# Clone and navigate to repository
git clone https://github.com/ptanmay143/virtual-therapist.git
cd virtual-therapist

# Create and activate virtual environment (recommended)
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# Train the model (if not already trained)
rasa train
```

**Requirements:**
- Python 3.8+
- [Rasa](https://rasa.com/docs/) - Conversational AI framework
- [Pandas](https://pandas.pydata.org/) - Data processing

## How It Works

The system uses a Rasa NLU pipeline with:

1. **Intent Classification** - DIETClassifier matches user input to FAQ categories
2. **Response Selection** - ResponseSelector retrieves the most relevant therapist answer
3. **Web Interface** - SocketIO-based chat widget for browser interaction

```
User Query → Rasa NLU → Intent Classification → Response Retrieval → Therapist Answer
```

**Data pipeline:**
- `data/raw.csv` contains question-answer pairs from therapist forums
- `main.ipynb` processes the CSV and generates Rasa training files
- `config.yml` defines the NLU pipeline (tokenizers, classifiers)
- `domain.yml` maps intents to responses

## API

The chatbot runs as a REST/SocketIO server on port 5005:

```bash
# Start server with custom options
rasa run --cors '*' --port 5005 --debug
# WARNING: --cors '*' allows any origin. For production, specify allowed domains:
# rasa run --cors 'https://yourdomain.com' --port 5005

# Test in interactive shell
rasa shell
```

**Configuration files:**

- `config.yml` - NLU pipeline (DIET classifier, response selector, 100 epochs)
- `domain.yml` - Response templates and actions (auto-generated from CSV)
- `credentials.yml` - Channel settings (REST, SocketIO)
- `data/nlu.yml` - Training examples (auto-generated from CSV)

**Adding new Q&A pairs:**

1. Add rows to `data/raw.csv` with columns: `questionTitle`, `questionText`, `answerText`
2. Run `jupyter nbconvert --to notebook --execute main.ipynb` to regenerate training files
3. Retrain with `rasa train`

## Limitations

- **FAQ-only responses** - No contextual conversation memory or multi-turn dialogues
- **No crisis detection** - Cannot assess emotional urgency or escalate to human support
- **Static responses** - Retrieves pre-written answers, doesn't generate novel text
- **Not HIPAA-compliant** - Data stored in plaintext without privacy measures

This is a proof-of-concept for accessible mental health information, not a replacement for professional therapy.

**Important:** If you're experiencing a mental health crisis:
- National Suicide Prevention Lifeline: 1-800-273-8255
- Crisis Text Line: Text HOME to 741741

## License

MIT License - See [LICENSE](LICENSE) for details.

Copyright © Tanmay Pachpande

## Background

This project demonstrates retrieval-based conversational AI for mental health FAQ. It's built on therapist forum data to make psychological guidance more accessible, but should complement (not replace) professional mental health services.

For production use, consider:
- Adding fallback handling for out-of-domain queries
- Implementing conversation context tracking
- Adding sentiment analysis for crisis detection
- Enhancing data privacy and HIPAA compliance
- Expanding the FAQ dataset with diverse mental health topics

See [Rasa documentation](https://rasa.com/docs/) for extending the NLU pipeline or deploying to production.
