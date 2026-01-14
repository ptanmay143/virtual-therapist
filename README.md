# Virtual Therapist

## Project Title

Virtual Therapist - Conversational AI FAQ Chatbot

## Overview

A Rasa-based conversational AI system designed to provide psychology-focused frequently asked questions (FAQ) responses. The system is trained on therapist forum discussions and provides accessible preliminary mental health guidance through a web chat interface. Using natural language understanding (NLU) and retrieval-based response selection, the chatbot matches user queries to curated therapist responses on mental health and relationship topics.

**Use Cases**: Mental health information, relationship advice, wellness guidance, preliminary psychological support

**Technology Stack**: Rasa NLU (conversational AI framework), Pandas (data processing), SocketIO (web chat)

## Architecture Overview

The system uses a retrieval-based question-answering architecture:

```
User Input (Chat Widget)
    ↓
Rasa NLU Pipeline
├─ WhitespaceTokenizer
├─ RegexFeaturizer
├─ LexicalSyntacticFeaturizer
├─ CountVectorsFeaturizer (word-level)
├─ CountVectorsFeaturizer (character n-grams 1-4)
├─ DIETClassifier (Intent Classification)
│   └─ 100 epochs of training
├─ EntitySynonymMapper
└─ ResponseSelector (FAQ Response Selection)
    └─ 100 epochs, retrieval_intent: faq
    ↓
Domain.yml (Curated Therapist Responses)
    ↓
RulePolicy (Single Rule: FAQ intent → utter_faq action)
    ↓
Chat Widget Display
```

**Data Flow**:
```
data/raw.csv (Source Q&A data)
    ↓ [main.ipynb: load_dataset()]
    ├─ Filter non-ASCII characters
    ├─ Clean text (remove quotes, colons, apostrophes)
    ├─ Group by questionTitle
    └─ Create (title, text, [answers]) tuples
    ↓
    ├─→ write_nlu()              ├─→ write_domain()
    │   ↓                        │   ↓
    │   data/nlu.yml             │   domain.yml
    │   (Training Examples)      │   (Responses + Actions)
    │   ↓                        │   ↓
    │   - intent: faq/0          │   responses:
    │     examples: |            │     utter_faq:
    │     - question title       │     - text: "..."
    │     - question text        │     utter_faq/1:
    │   - intent: faq/1          │     - text: "..."
    │     ...                    │
    ↓ [config.yml: NLU pipeline] ↓
Rasa Train
    ↓
Trained Models & Policies
    ↓
rasa run --cors '*' --port 5005
    ↓
REST/SocketIO Server
    ↓
Web Chat Widget (index.html)
```

**Components**:
- **config.yml**: NLU pipeline definition (tokenizers, feature extractors, classifiers)
- **domain.yml**: Intent definitions, responses, actions
- **data/nlu.yml**: Training examples (generated from raw.csv)
- **data/raw.csv**: Source Q&A dataset (questionTitle, questionText, answerText)
- **main.ipynb**: Data processing pipeline converting CSV to Rasa format
- **index.html**: Web chat widget UI
- **credentials.yml**: Channel configuration (REST, SocketIO)

## Installation and Setup

### Prerequisites

- Python 3.8 or higher
- pip (Python package manager)
- Virtual environment (recommended)
- Git

### Step 1: Clone Repository

```bash
git clone https://github.com/ptanmay143/virtual-therapist.git
cd virtual-therapist
```

### Step 2: Create Virtual Environment (Recommended)

```bash
# Create virtual environment
python -m venv venv

# Activate virtual environment
# Windows:
venv\Scripts\activate
# macOS/Linux:
source venv/bin/activate
```

### Step 3: Install Dependencies

```bash
pip install -r requirements.txt
```

**Installed packages:**
- `rasa` - Conversational AI framework
- `pandas` - Data manipulation and CSV processing

### Step 4: Generate Configuration Files

If `data/raw.csv` is updated, regenerate NLU and domain files:

```bash
# Option 1: Run with Jupyter
jupyter notebook main.ipynb
# Execute all cells

# Option 2: Run with nbconvert
jupyter nbconvert --to notebook --execute main.ipynb

# Option 3: Run with Python (if notebook format allows)
python -c "import nbformat; exec(open('main.ipynb').read())"
```

**Generated Files**:
- `data/nlu.yml` - NLU training examples from raw.csv
- `domain.yml` - Domain responses from raw.csv

## Configuration

### Rasa NLU Pipeline (config.yml)

```yaml
language: en
pipeline:
  - WhitespaceTokenizer           # Tokenization
  - RegexFeaturizer               # Pattern-based features
  - LexicalSyntacticFeaturizer    # Syntactic analysis
  - CountVectorsFeaturizer        # Word-level vectors
  - CountVectorsFeaturizer        # Character n-grams (1-4)
    analyzer: char_wb
    min_ngram: 1
    max_ngram: 4
  - DIETClassifier                # Intent/entity classification
    epochs: 100
    constrain_similarities: true
  - EntitySynonymMapper           # Synonym resolution
  - ResponseSelector              # FAQ response selection
    epochs: 100
    retrieval_intent: faq
    constrain_similarities: true
```

**Optional: Enable Fallback**:
```yaml
- name: FallbackClassifier
  threshold: 0.3
  ambiguity_threshold: 0.1
```

Handles out-of-domain queries gracefully.

### Chat Widget Configuration (index.html)

```javascript
socketUrl: "http://localhost:5005"  // Rasa server address
// Adjust if running on different host/port
```

### Channel Configuration (credentials.yml)

```yaml
rest:              # REST API endpoint
socketio:
  user_message_evt: user_uttered    # User message event
  bot_message_evt: bot_uttered      # Bot message event
  session_persistence: true         # Persist conversation state
```

## Usage

### Step 1: Start Rasa Server

```bash
# Run with CORS enabled (required for web chat)
rasa run --cors '*' --port 5005

# Options:
# --cors '*'              Enable cross-origin requests
# --port 5005             Server port (default)
# --debug                 Enable debug logging
```

Server accessible at: `http://localhost:5005`

### Step 2: Access Chat Widget

Open `index.html` in web browser:
```bash
# Windows
start index.html

# macOS
open index.html

# Linux
xdg-open index.html
```

Or embed in another web page:
```html
<iframe src="index.html" width="400" height="600"></iframe>
```

### Step 3: Example Interactions

**User**: "My husband cheated on me and I'm struggling to forgive him"
**Bot**: [Retrieves matching FAQ response with therapist advice on infidelity]

**User**: "Am I gay?"
**Bot**: [Returns response on sexual identity exploration]

**User**: "I'm having suicidal thoughts"
**Bot**: [Returns response OR escalates with crisis hotline numbers]

## Dependencies

| Package | Version | Purpose |
|---------|---------|---------|
| `rasa` | Latest | Conversational AI framework |
| `pandas` | Latest | Data processing (CSV manipulation) |

**Install all dependencies**:
```bash
pip install -r requirements.txt
```

**Transitive Dependencies** (installed automatically):
- spacy - NLP library
- scikit-learn - ML algorithms
- tensorflow - Deep learning
- numpy, scipy - Scientific computing
- And others required by Rasa

## Development and Testing

### Code Organization

**main.ipynb** - Data processing pipeline:
- `is_ascii(s)` - Validates ASCII-encodable strings
- `filter(s)` - Cleans text (removes special characters, non-ASCII)
- `load_dataset()` - Reads raw.csv and structures Q&A
- `write_nlu()` - Generates data/nlu.yml (training examples)
- `write_domain()` - Generates domain.yml (responses)
- `main()` - Orchestrates entire pipeline

**config.yml** - NLU pipeline configuration:
- Tokenizer selection
- Feature extractor configuration
- Classifier epochs and thresholds
- Response selector training

**domain.yml** - Domain definition:
- Intent list
- Response mapping (generated)
- Action definitions
- Policy configuration

### Adding New Q&A Pairs

1. Add rows to `data/raw.csv`:
   - Column `questionTitle` - Short question title
   - Column `questionText` - Full question text
   - Column `answerText` - Therapist response

2. Regenerate configuration files:
   ```bash
   jupyter nbconvert --to notebook --execute main.ipynb
   ```

3. Retrain the model:
   ```bash
   rasa train
   ```

4. Restart the server:
   ```bash
   rasa run --cors '*'
   ```

### Extending NLU Pipeline

To add new classifier or feature extractor:

1. Edit `config.yml` pipeline section
2. Run `rasa train` to retrain model
3. Test with `rasa shell` or web widget

Example: Adding FallbackClassifier
```yaml
- name: FallbackClassifier
  threshold: 0.3
  ambiguity_threshold: 0.1
```

### Testing

**Interactive Shell Testing**:
```bash
rasa shell
# Type user queries and observe bot responses
```

**Manual Widget Testing**:
1. Start server: `rasa run --cors '*'`
2. Open index.html
3. Type test messages and verify correct responses

**Recommended Test Cases**:
- Intent classification accuracy (should classify to correct FAQ)
- Response retrieval quality
- Edge cases (non-English input, ambiguous queries, out-of-domain)
- Web widget CORS and message delivery
- SocketIO session persistence

## Deployment

### Option 1: Local Development

```bash
rasa run --cors '*'
# Access at http://localhost:5005
```

### Option 2: Docker Containerization

Create `Dockerfile`:
```dockerfile
FROM python:3.8-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
CMD ["rasa", "run", "--cors", "'*'"]
```

Build and run:
```bash
docker build -t virtual-therapist .
docker run -p 5005:5005 virtual-therapist
```

### Option 3: Cloud Deployment

**Heroku**:
1. Create `Procfile`: `web: rasa run --cors '*' --port $PORT`
2. Deploy via Git push or CI/CD

**AWS/GCP**:
1. Containerize with Docker
2. Push to container registry
3. Deploy to Cloud Run, ECS, or App Engine

**Production Considerations**:
- Use HTTPS/SSL for security
- Configure CORS with specific domain (not '*')
- Set up structured logging
- Monitor server health and performance
- Rate limiting to prevent abuse
- Enable backup of trained models

## Limitations and Future Improvements

### Limitations

| Issue | Impact |
|-------|--------|
| FAQ-only responses | Only handles predefined intents |
| No contextual memory | Stateless responses (no conversation history) |
| No sentiment analysis | Cannot assess emotional urgency |
| Limited entity recognition | No extraction of specific entities |
| Static responses | Cannot generate novel responses |
| No user personalization | All users receive identical responses |
| No privacy measures | Q&A data stored plaintext (not HIPAA-compliant) |
| ASCII-only filtering | Loses nuance from non-English text |
| No logging/analytics | Cannot analyze user interactions |
| Single domain only | Cannot handle non-FAQ queries |

### Future Improvements

**High Priority**:
- [ ] Implement Fallback Handling (gracefully handle out-of-domain)
- [ ] Add Contextual Conversation Flow (multi-turn dialogues)
- [ ] Add Unit & Integration Tests (pytest framework)
- [ ] Comprehensive Logging & Monitoring (track accuracy, performance)

**Medium Priority**:
- [ ] Safety & Ethics (crisis detection, escalation to human)
- [ ] Data Enhancement (expand FAQ dataset, multilingual support)
- [ ] Deployment & Scaling (Docker, CI/CD, load balancing)
- [ ] User Experience (typing indicators, better UI/UX)

**Low Priority**:
- [ ] Advanced NLU (LLM-based generation, sentiment analysis)
- [ ] Generalization (support other domains, modular structure)
- [ ] Analytics (conversation tracking, topic modeling)

## Contribution Guidelines

1. Fork the repository
2. Create feature branch: `git checkout -b feature/your-feature`
3. Add/update tests for changes
4. Update README.md with changes
5. Submit pull request with clear description

### Code Style

- Follow PEP 8 for Python
- Use descriptive variable/function names
- Add comments for complex logic
- Keep functions small and focused

### Reporting Issues

- Describe issue clearly
- Include steps to reproduce
- Attach logs or screenshots
- Suggest a fix if possible

## License

MIT License - See [LICENSE](LICENSE) file for complete terms.

**Copyright**: Tanmay Pachpande

**Permissions**: Commercial use, modification, distribution, private use

**Conditions**: License and copyright notice must be included

**Limitations**: No liability or warranty

## Support & Resources

- **Rasa Documentation**: https://rasa.com/docs/
- **Rasa Community Forum**: https://forum.rasa.com/
- **Mental Health Resources**:
  - National Suicide Prevention Lifeline: 1-800-273-8255
  - Crisis Text Line: Text HOME to 741741
  - SAMHSA Helpline: 1-800-662-4357
