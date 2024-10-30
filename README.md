# PDF_Question_Answering
This project is a backend service for PDF-based question-answering. Users can upload PDFs, then ask questions via WebSocket to get real-time answers based on the document’s content. Built with FastAPI, PyMuPDF, and NLP models, it includes Redis-based rate limiting for efficient handling of requests, making it ideal for interactive document analysis

# PDF Question-Answering Backend Service

## Overview

This backend service allows users to upload PDF documents, ask questions about their content in real-time via WebSocket, and receive relevant answers. Rate limiting is applied to manage usage, and session-based context is maintained for follow-up questions.

## Features

- **PDF Upload**: Upload PDF documents, extract text, and store metadata.
- **WebSocket Question Answering**: Real-time question answering with session-based context.
- **Rate Limiting**: Prevents abuse by limiting the number of requests per user.

## Technologies Used

- **FastAPI**: Provides the backend framework.
- **Redis**: Supports rate limiting and session management.
- **PyMuPDF**: Extracts text from PDFs.
- **LangChain and LlamaIndex**: Processes natural language questions based on PDF content.

## Setup Instructions

### Prerequisites

- **Python 3.8+**
- **Redis server** (for rate limiting)

### Installation

1. **Clone the repository**:
    ```bash
    git clone https://github.com/username/pdf-qa-service.git
    cd pdf-qa-service
    ```

2. **Install dependencies**:
    ```bash
    pip install -r requirements.txt
    ```

3. **Start Redis server**:
    ```bash
    redis-server
    ```

4. **Run the FastAPI application**:
    ```bash
    uvicorn app.main:app --reload
    ```

### API Documentation

After starting the server, the interactive API documentation is available at `http://127.0.0.1:8000/docs`.

### Endpoints

1. **POST /upload-pdf/**: Upload a PDF document.
    - Request: Upload a PDF file.
    - Response: Returns a JSON response containing the file metadata.

2. **WebSocket /ws/question/**: Connect to WebSocket for real-time Q&A.
    - Message Format: `"{document_id}|{question}"`.
    - Response: Returns answers based on PDF content, with rate limiting applied.

### Architectural Overview

The service uses FastAPI to manage HTTP requests and WebSocket connections. PDF text is extracted using PyMuPDF and stored in an SQLite database. The NLP processing for question answering is handled by LangChain and LlamaIndex, which allow for context-aware responses.

---

### 3. **Testing Script**

The testing script covers each endpoint and functionality, including WebSocket, PDF upload, and rate limiting.

#### **tests/test_main.py**

```python
from fastapi.testclient import TestClient
from app.main import app
import pytest

client = TestClient(app)

def test_upload_pdf():
    with open("sample.pdf", "rb") as pdf_file:
        response = client.post("/upload-pdf/", files={"file": pdf_file})
    assert response.status_code == 200
    assert "filename" in response.json()

def test_websocket_question():
    with client.websocket_connect("/ws/question/") as websocket:
        websocket.send_text("1|What is the travel date?")
        data = websocket.receive_text()
        assert "date" in data  # Assuming the PDF contains a date field

def test_rate_limiting():
    with client.websocket_connect("/ws/question/") as websocket:
        for _ in range(12):  # Exceed the rate limit of 10 messages/min
            websocket.send_text("1|What is the PNR number?")
        response = websocket.receive_text()
        assert "Rate limit exceeded" in response



# run test with:
pytest tests/test_main.py
