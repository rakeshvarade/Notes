# Artifacts in ADK (Agent Development Kit)

Reference: [https://adk.dev/artifacts/](https://adk.dev/artifacts/)

---

## 1. Simple Example

Imagine an AI agent designed to generate professional headshots. When the agent finishes creating an image, it doesn't just send back text; it creates an **Artifact**.

```python
# Simple Python snippet to save a generated image as an artifact
from google.genai import types

# 1. Define the binary data (the image)
image_bytes = b'\x89PNG\r\n...' # Placeholder for raw image data

# 2. Wrap it in a Part object with a MIME type
image_artifact = types.Part.from_bytes(data=image_bytes, mime_type="image/png")

# 3. Save it via the context
await context.save_artifact(filename="user_headshot.png", artifact=image_artifact)
```

---

## 2. Detailed Explanation

In the **Agent Development Kit (ADK)**, an **Artifact** is a mechanism for managing **named, versioned binary data**. While standard session state is great for text strings or simple variables, Artifacts are the "file system" of your agent.

### Core Components:
* **Binary Data (Blob):** Artifacts handle raw bytes—images, PDFs, audio files, or spreadsheets.
* **MIME Type:** Every artifact identifies its format (e.g., `application/pdf`), which tells the system how to read the data later.
* **Versioning:** ADK automatically versions artifacts. If you save `report.pdf` twice, the system stores both versions, allowing you to "roll back" or compare them.
* **Namespacing:** Artifacts can be scoped to a single **Session** (temporary) or a **User** (persistent across different conversations).

### Analogy
Think of **Session State** like a **Post-it note** on your desk; it's great for quick reminders (like a user's name), but it's small and easily lost. An **Artifact** is like a **Filing Cabinet**. It holds thick folders (large binary files), every document is labeled (filename), and every time you update a document, you keep the old version underneath (versioning).


---

## 3. Mid-Complex Example

In a more technical scenario, you might want to **load** a specific version of an artifact and process it. This is common when an agent needs to reference a document a user uploaded earlier in the session.

```python
# Loading and processing a specific version of a PDF artifact
async def process_report(context):
    filename = "monthly_analysis.pdf"
    
    try:
        # Retrieve the latest version of the artifact
        artifact = await context.load_artifact(filename=filename)
        
        if artifact and artifact.inline_data:
            pdf_data = artifact.inline_data.data
            mime_type = artifact.inline_data.mime_type
            print(f"Processing {filename} ({len(pdf_data)} bytes) as {mime_type}")
            
            # Example: Load version 0 specifically if needed
            # old_version = await context.load_artifact(filename=filename, version=0)
            
    except ValueError:
        print("Artifact Service not configured!")
```

---

## 4. Real-Time Example

**Scenario: Autonomous Legal Assistant**
In a real-world legal tech application, an ADK-powered agent might perform the following:
1.  **Input:** The user uploads a 50-page contract (Saved as a `user:contract.pdf` artifact).
2.  **Processing:** The agent analyzes the contract and generates a summary report.
3.  **Storage:** The summary is saved as a new artifact `summary_v1.pdf`.
4.  **Refinement:** If the user asks to change the tone, the agent generates a new version, automatically saved as `summary_v2.pdf`.
5.  **Retrieval:** A week later, the user returns. Because the artifact was saved with the `user:` prefix, the agent can still retrieve the contract and its summaries from **Google Cloud Storage (GCS)**.

---

## 5. Artifacts Cheatsheet

| Feature | Description |
| :--- | :--- |
| **`save_artifact`** | Saves binary data. Automatically creates a new version if the name exists. |
| **`load_artifact`** | Retrieves a `Part` object. Defaults to the latest version. |
| **`list_artifacts`** | Returns a list of all filenames currently stored in the scope. |
| **`InMemoryArtifactService`** | Best for **testing/prototyping**. Data is lost when the app stops. |
| **`GcsArtifactService`** | Best for **production**. Uses Google Cloud Storage for persistent files. |
| **`user:` prefix** | Add this to a filename (e.g., `user:profile.jpg`) to persist it across sessions. |
| **MIME Types** | Always specify (e.g., `image/jpeg`, `text/csv`) to ensure correct interpretation. |

---

*Reference: [https://adk.dev/artifacts/](https://adk.dev/artifacts/)*