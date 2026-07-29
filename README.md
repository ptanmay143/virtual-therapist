<h1 align="center">
	<a href="https://github.com/ptanmay143/virtual-therapist">
		<img src="docs/images/logo.svg" alt="Logo" width="100" height="100">
	</a>
</h1>

<div align="center">
	Virtual Therapist
	<br />
	<a href="#about"><strong>Explore the screenshots »</strong></a>
	<br />
	<br />
	<a href="https://github.com/ptanmay143/virtual-therapist/issues/new?assignees=&labels=bug&template=01_BUG_REPORT.md&title=bug%3A+">Report a Bug</a>
	·
	<a href="https://github.com/ptanmay143/virtual-therapist/issues/new?assignees=&labels=enhancement&template=02_FEATURE_REQUEST.md&title=feat%3A+">Request a Feature</a>
	·
	<a href="https://github.com/ptanmay143/virtual-therapist/issues/new?assignees=&labels=question&template=04_SUPPORT_QUESTION.md&title=support%3A+">Ask a Question</a>
</div>

<div align="center">
<br />

[![Project license](https://img.shields.io/github/license/ptanmay143/virtual-therapist.svg?style=flat-square)](LICENSE)
[![Pull Requests welcome](https://img.shields.io/badge/PRs-welcome-ff69b4.svg?style=flat-square)](https://github.com/ptanmay143/virtual-therapist/issues?q=is%3Aissue+is%3Aopen+label%3A%22help+wanted%22)
[![code with love by ptanmay143](https://img.shields.io/badge/%3C%2F%3E%20with%20%E2%99%A5%20by-ptanmay143-ff1414.svg?style=flat-square)](https://github.com/ptanmay143)

</div>

<details open="open">
<summary>Table of Contents</summary>

- [About](#about)
  - [Built With](#built-with)
- [Getting Started](#getting-started)
  - [Prerequisites](#prerequisites)
  - [Installation](#installation)
- [Usage](#usage)
- [Roadmap](#roadmap)
- [Support](#support)
- [Project Assistance](#project-assistance)
- [Contributing](#contributing)
- [Authors & contributors](#authors--contributors)
- [Security](#security)
- [License](#license)
- [Acknowledgements](#acknowledgements)

</details>

---

## About

Virtual Therapist is a retrieval-based conversational assistant built with Rasa. It maps user questions to predefined FAQ intents and returns curated text responses sourced from therapist-style Q&A data.

The project is designed for informational mental-health Q&A exploration, not for diagnosis or emergency intervention. It emphasizes deterministic response retrieval over generative model behavior, making output reproducible and directly traceable to curated responses in the training/domain files.

Runtime behavior combines a Rasa server (`rasa run`) and a browser widget (`index.html`) loaded from the `rasa-webchat` CDN. The widget connects to the local Socket.IO endpoint and sends user utterances for intent matching and response selection.

The training data and response templates are substantial. `data/nlu.yml` defines many `faq/*` intents with examples, `domain.yml` contains corresponding `utter_faq/*` responses, and `rules.yml` routes all `faq` intents to `utter_faq` through `RulePolicy` behavior.

<details>
<summary>Screenshots</summary>
<br>

Add screenshots under `docs/images/` and update this section if needed.

|                               Chat Widget                               |                               Conversation                               |
| :---------------------------------------------------------------------: | :----------------------------------------------------------------------: |
| <img src="docs/images/screenshot.png" title="Chat Widget" width="100%"> | <img src="docs/images/screenshot.png" title="Conversation" width="100%"> |

</details>

### Built With

- **Python** — runtime for Rasa and data tooling.
- **Rasa** — NLU, retrieval intent handling, and chat server runtime.
- **Pandas** — data processing dependency (used in notebook-based preparation flow).
- **Rasa Webchat (CDN)** — browser chat widget integration.
- **YAML configuration** — pipeline/policy/domain/rules declarations.

---

## Getting Started

Setup consists of Python dependency installation, model training, server startup, and launching the browser widget page.

### Prerequisites

- **Python 3.x** compatible with your selected Rasa release.
- **pip** for dependency installation.
- **Rasa CLI available** (`rasa --version`).
- Optional: **Jupyter** if you plan to execute `main.ipynb` for data regeneration.

### Installation

1. Clone repository.

```bash
git clone https://github.com/ptanmay143/virtual-therapist.git
```

2. Enter project folder.

```bash
cd virtual-therapist
```

3. Create and activate virtual environment.

```bash
python -m venv venv
venv\Scripts\activate
```

4. Install Python dependencies.

```bash
pip install -r requirements.txt
```

5. Train Rasa model.

```bash
rasa train
```

6. Start chatbot server.

```bash
rasa run --cors "*" --port 5005
```

7. Open web widget page.

```bash
python -m http.server 8000
```

8. Verify setup.

```text
Open http://localhost:8000/index.html and check that the widget connects
to http://localhost:5005 and returns FAQ responses.
```

### Environment Variables

Current repository configuration does not require environment variables.

| Variable | Required | Default | Description                                                | Example Value |
| -------- | -------- | ------- | ---------------------------------------------------------- | ------------- |
| None     | No       | N/A     | Runtime config is declared in YAML and command-line flags. | N/A           |

---

## Usage

Train model:

```bash
rasa train
```

Run API/chat server:

```bash
rasa run --cors "*" --port 5005 --debug
```

Test in shell:

```bash
rasa shell
```

Serve widget page:

```bash
python -m http.server 8000
```

Open:

```text
http://localhost:8000/index.html
```

Core config behavior:

- `config.yml`: pipeline includes `WhitespaceTokenizer`, `RegexFeaturizer`, two `CountVectorsFeaturizer` stages, `DIETClassifier` (100 epochs), and `ResponseSelector` with `retrieval_intent: faq`.
- `rules.yml`: maps intent `faq` to action `utter_faq`.
- `credentials.yml`: enables `rest` and `socketio` with `user_uttered`/`bot_uttered` events.

Data-refresh workflow (if updating source Q&A):

1. Update `data/raw.csv`.
2. Execute notebook pipeline in `main.ipynb` to regenerate training/domain content.
3. Re-train model with `rasa train`.

API/channel summary:

- Socket endpoint: `http://localhost:5005` (from `index.html` widget config).
- REST channel enabled in `credentials.yml`.

---

## Roadmap

See the [open issues](https://github.com/ptanmay143/virtual-therapist/issues) for a full list of proposed features and known bugs.

- [Top Feature Requests](https://github.com/ptanmay143/virtual-therapist/issues?q=label%3Aenhancement+is%3Aopen+sort%3Areactions-%2B1-desc) (Add your votes using the 👍 reaction)
- [Top Bugs](https://github.com/ptanmay143/virtual-therapist/issues?q=is%3Aissue+is%3Aopen+label%3Abug+sort%3Areactions-%2B1-desc) (Add your votes using the 👍 reaction)
- [Newest Bugs](https://github.com/ptanmay143/virtual-therapist/issues?q=is%3Aopen+is%3Aissue+label%3Abug)

Potential future direction includes stronger fallback handling, conversation-state support beyond single-turn FAQ retrieval, better quality control for response content, and production-grade safety/privacy hardening.

---

## Support

Reach out to the maintainer at one of the following places:

- [GitHub issues](https://github.com/ptanmay143/virtual-therapist/issues/new?assignees=&labels=question&template=04_SUPPORT_QUESTION.md&title=support%3A+)
- Contact options listed on [this GitHub profile](https://github.com/ptanmay143)

---

## Project Assistance

If you want to say **thank you** or support active development of Virtual Therapist:

- Add a [GitHub Star](https://github.com/ptanmay143/virtual-therapist) to the project.
- Contribute safer, better-curated mental-health Q&A datasets.
- Share reproducible evaluations of intent/response quality.

Together, we can make Virtual Therapist **better**!

---

## Contributing

First off, thanks for taking the time to contribute! Contributions are what make the open-source community such an amazing place to learn, inspire, and create.

Suggested workflow:

1. Fork and branch from `master`.
2. Apply targeted changes to data, config, or integration code.
3. Re-train and validate with `rasa shell` and webchat.
4. Document any dataset or behavior changes in your pull request.

No dedicated `docs/CONTRIBUTING.md` file exists currently.

---

## Authors & Contributors

The original setup of this repository is by [Tanmay Pachpande](https://github.com/ptanmay143).

For a full list of all authors and contributors, see [the contributors page](https://github.com/ptanmay143/virtual-therapist/contributors).

---

## Security

Virtual Therapist follows good practices of security, but 100% security cannot be assured. Virtual Therapist is provided **"as is"** without any **warranty**. Use at your own risk.

Important safety note: this project is an informational FAQ assistant and is not a substitute for licensed professional care, emergency response, or crisis intervention. No dedicated `docs/SECURITY.md` exists currently.

---

## License

This project is licensed under the **MIT License**.

See [LICENSE](LICENSE) for more information.

---

## Acknowledgements

- Rasa open-source community and documentation ecosystem.
- Contributors to mental-health Q&A datasets and annotation workflows.
- Python data and NLP tooling communities.

<!-- Generated by README_GENERATOR_PROMPT v0.1 -->
