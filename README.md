# AI Insights: Architectural review of a multi-instance neural video search engine

[![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.18827669.svg)](https://doi.org/10.5281/zenodo.18827669)

> **Note:** This repository provides an architectural overview and technical case study for my AI Insights prototype. The source code is currently maintained in a private repository.

## Abstract

For fellow journalists, historical researchers, and documentarians, locating specific events hidden within massive archives of unlabelled footage represents a significant operational bottleneck. Video data is inherently opaque to standard search methodologies. Historically, retrieving a specific clip required a human archivist to manually watch, log, and assign text metadata to the file. This process is highly subjective, financially expensive, and entirely unscalable (Velazquez, 2018).

This AI Insights project was developed to bypass traditional metadata entirely. By leveraging multimodal neural search, the application maps the actual visual and audio contents of a video into searchable vector data. Rather than returning a single, isolated match, the system locates every relevant occurrence across an entire archive, applies a strict temporal filter to remove redundant frames, and routes the user to the exact second the event occurs via a custom interface. This document outlines the system architecture, core engineering decisions, and key technical learnings derived from the prototype.

## Core capabilities

* **Multimodal semantic retrieval:** The engine understands the contextual meaning behind natural language queries. A researcher can search for "heavy manufacturing equipment" or "wildlife in the canopy," and the system will cross-reference this text against the visual and audio vectors of the media files. This eliminates the reliance on exact keyword matches or the assumption that an archivist applied the correct manual tag years prior.
* **Temporal deduplication and clustering:** Video search presents a unique challenge because it is a time-series medium. A single continuous camera pan might trigger dozens of positive machine matches within a few seconds. The system uses a custom backend algorithm to group matches that occur within a 10-second temporal window. This prevents the user interface from being overwhelmed by repetitive clips of the exact same scene, significantly reducing cognitive load.
* **Targeted temporal navigation:** The frontend is designed around a split-screen interface that auto-plays the primary match immediately while generating a clickable, curated list of alternative instances. This allows researchers to jump instantly between exact moments across entirely different archival files without the need for manual scrubbing or timeline manipulation.

## Technology stack

The application deliberately separates the user interface, data processing logic, and artificial intelligence analysis into distinct environments. This decoupled architecture ensures application stability, protects secure credentials, and allows for independent scaling.

* **Frontend environment:** React (JavaScript) – A responsive, dark-themed interface built specifically to facilitate the review of video evidence without visual distractions or unnecessary user interface elements.
* **Backend environment:** FastAPI (Python) – A fast, asynchronous API layer that manages data payloads, enforces network rate limits, and executes the heavy deduplication algorithms.
* **Neural analysis engine:** Twelve Labs API – A foundational multimodal model that handles the intensive computational work of visual and audio scanning, vector embedding, and search retrieval.

## System flow and architecture

<img width="1113" height="667" alt="image" src="https://github.com/user-attachments/assets/8525fb53-c95a-481e-a5cd-2d864e22092d" />


1. **Query initiation:** The user inputs a natural language search string into the React frontend.
2. **Secure proxying:** The FastAPI backend receives the request and securely passes it to the Twelve Labs neural engine, keeping all API keys and index identifiers hidden from the client browser.
3. **Vector analysis:** The neural engine scans the pre-indexed video database, comparing the search query against the visual and audio embeddings of the archive.
4. **Algorithmic curation:** The engine returns a raw, unstructured list of timecoded matches. The backend intercepts this machine data and applies a 10-second global filter, discarding overlapping frames to ensure only distinct, unique moments are retained.
5. **Media resolution:** For the top five unique instances, the backend requests the exact video source URLs and necessary access tokens from the database.
6. **Interface rendering:** The highly curated dataset is returned to the frontend. The application updates the main video player state (skipping straight to the exact second of the primary match) and populates the sidebar with the remaining instances for immediate human review.

## Key engineering challenges

### 1. API obfuscation and secure proxying
Attempting to run search logic directly from a browser application is a severe security risk. It exposes proprietary database structures and billing credentials to the public network. The FastAPI backend was implemented as a mandatory secure middleman. It intercepts the user's intent, constructs the correct data payloads server-side, and communicates with the neural engine privately, ensuring complete security for the underlying infrastructure.

### 2. Algorithmic temporal clustering
There is a profound difference between mathematical machine accuracy and practical human utility. Neural engines view video as a continuous stream of individual frames. Consequently, they often return overlapping results (for example, registering a match at 316 seconds, 317 seconds, and 318 seconds for the exact same physical event). Presenting this raw data to a user renders the tool practically useless. The backend required a custom Python algorithm to intercept this machine output, mathematically compare the timestamps, and cluster them into single, human-readable instances before updating the screen.

### 3. Asynchronous media state management
Modern web browsers are notoriously rigid when handling large video files dynamically. Appending a `#t={timecode}` parameter to an HTML5 video source is the standard method for deep-linking into a video. However, executing this via React state to ensure immediate, buffer-free jumping between search results requires precise handling of the component lifecycle. If the state updates too slowly, the video will play from the beginning; if it updates too quickly, the browser may throw a media fetching error. Managing these transitions smoothly, especially when switching between entirely different video files, required strict dependency management within the React hooks.

## Key project learnings

### 1. The prerequisite of asynchronous vector indexing
Unlike standard text search, a neural video engine cannot scan a raw MP4 file on the fly. To achieve rapid search results, the footage must first be uploaded, processed, and mapped into a multidimensional vector index. This ingestion phase is asynchronous and computationally heavy. A core learning of this project is that video search is fundamentally a two-step process: building a robust data ingestion pipeline is an absolute prerequisite before any front-end search capabilities can be developed.

### 2. Reconciling machine precision with human utility
Artificial intelligence models are highly precise, but they lack human context. As discovered during the initial testing phases, a system that returns 50 highly accurate matches of the same ten-second camera pan is technically correct but practically flawed. Building an intermediary logic layer to curate, filter, and structure the raw mathematical data was essential to turn an impressive AI capability into a genuinely useful research tool.

### 3. The necessity of decoupled state and logic architectures
While it is tempting in early prototyping to build 'monolithic' applications where the frontend handles both user interaction and data processing, this approach fails quickly when dealing with large datasets like video. A decoupled architecture allows the lightweight Python backend to handle the heavy computational filtering and data transformation, ensuring the React frontend remains fast, responsive, and dedicated entirely to the user experience.

---
*Jose Velazquez (MA) – 2026*
