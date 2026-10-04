---
layout: post
title: "TRACE: Spatial Triage Board (Interactive Mockup)"
date: 2026-10-04
categories: [architecture, UI]
---

<p style="color: #6b7280; font-style: italic;">Note: This is a frontend UI mockup. The WebAssembly engine is simulated for layout testing.</p>

<style>
    /* TRACE UI Styles */
    .trace-container { font-family: system-ui, -apple-system, sans-serif; margin-top: 2rem; }
    .trace-dropzone { 
        border: 2px dashed #9ca3af; border-radius: 8px; padding: 40px; 
        text-align: center; background: #f3f4f6; cursor: pointer; transition: background 0.2s;
    }
    .trace-dropzone:hover { background: #e5e7eb; }
    
    .trace-workspace { display: flex; gap: 20px; margin-top: 20px; min-height: 500px; }
    .trace-col { 
        flex: 1; padding: 15px; border-radius: 8px; border: 1px solid #e5e7eb; 
        display: flex; flex-direction: column; gap: 10px;
    }
    .trace-col h3 { margin-top: 0; font-size: 1.1rem; padding-bottom: 10px; border-bottom: 2px solid #e5e7eb; }
    
    .col-contract { background: #f9fafb; }
    .col-trunk { background: #f0fdf4; border-color: #bbf7d0; }
    .col-quarantine { background: #fef2f2; border-color: #fecaca; }

    .node { padding: 12px; border-radius: 6px; font-size: 0.9rem; line-height: 1.4; }
    .node-ghost { background: #e5e7eb; color: #4b5563; border: 1px dashed #9ca3af; }
    .node-safe { background: #d1fae5; color: #065f46; border: 1px solid #34d399; }
    .node-danger { background: #fee2e2; color: #991b1b; border: 1px solid #f87171; }
    
    .badge { 
        font-size: 0.7rem; font-weight: bold; text-transform: uppercase; 
        display: inline-block; margin-bottom: 6px; padding: 2px 6px; border-radius: 4px;
    }
    .badge-safe { background: #059669; color: white; }
    .badge-danger { background: #dc2626; color: white; }

    .trace-btn {
        display: none; width: 100%; margin-top: 20px; padding: 15px;
        background: #2563eb; color: white; border: none; border-radius: 8px;
        font-size: 1rem; font-weight: bold; cursor: pointer; transition: background 0.2s;
    }
    .trace-btn:hover { background: #1d4ed8; }
</style>

<div class="trace-container">
    <!-- Ingestion Zone -->
    <div id="dropzone" class="trace-dropzone">
        <h3 style="margin:0; color: #374151;">Drag & Drop .docx File Here</h3>
        <p id="dropzone-text" style="color: #6b7280; margin-top: 8px;">or click to browse</p>
        <input type="file" id="fileInput" accept=".docx" style="display: none;">
    </div>

    <!-- Triage Workspace -->
    <div class="trace-workspace">
        <div class="trace-col col-contract" id="col-contract">
            <h3>Template Standard</h3>
            <div class="node node-ghost"><strong>3.0 Responsibilities</strong><br>Table: Role | Responsibility</div>
            <div class="node node-ghost"><strong>5.0 Definitions</strong><br>Table: Term | Definition</div>
            <div class="node node-ghost"><strong>6.0 Procedure</strong><br>Table: Step | Role | Procedure</div>
        </div>
        
        <div class="trace-col col-trunk" id="col-trunk">
            <h3>Conforming Trunk</h3>
            <!-- Populated via JS -->
        </div>
        
        <div class="trace-col col-quarantine" id="col-quarantine">
            <h3>Quarantine Zone</h3>
            <!-- Populated via JS -->
        </div>
    </div>

    <!-- Return Trip Action -->
    <button id="downloadBtn" class="trace-btn">Download Annotated .docx (Return Trip)</button>
</div>

<script>
    const dropzone = document.getElementById('dropzone');
    const fileInput = document.getElementById('fileInput');
    const dropzoneText = document.getElementById('dropzone-text');
    const colTrunk = document.getElementById('col-trunk');
    const colQuarantine = document.getElementById('col-quarantine');
    const downloadBtn = document.getElementById('downloadBtn');

    // Click to upload
    dropzone.addEventListener('click', () => fileInput.click());

    // Drag and drop handlers
    dropzone.addEventListener('dragover', (e) => { e.preventDefault(); dropzone.style.background = '#e5e7eb'; });
    dropzone.addEventListener('dragleave', () => dropzone.style.background = '#f3f4f6');
    dropzone.addEventListener('drop', (e) => {
        e.preventDefault();
        dropzone.style.background = '#f3f4f6';
        if (e.dataTransfer.files.length) triggerMockSimulation(e.dataTransfer.files[0].name);
    });
    fileInput.addEventListener('change', (e) => {
        if (e.target.files.length) triggerMockSimulation(e.target.files[0].name);
    });

    function triggerMockSimulation(fileName) {
        dropzoneText.innerHTML = `<strong>Selected:</strong> ${fileName} <br><em>Simulating WASM DAG Extraction...</em>`;
        
        // Simulate processing delay
        setTimeout(() => {
            dropzoneText.innerHTML = `<strong>Selected:</strong> ${fileName} <br><span style="color: #059669;">✔ Triage Complete</span>`;
            
            // Populate Conforming Trunk
            colTrunk.innerHTML = `
                <h3>Conforming Trunk</h3>
                <div class="node node-safe">
                    <span class="badge badge-safe">Mapped: 3.0</span><br>
                    <strong>Cook:</strong> Follow this BP when baking apple pies...
                </div>
                <div class="node node-safe">
                    <span class="badge badge-safe">Mapped: 6.1.1</span><br>
                    <strong>Cook:</strong> Mix dough until just coming together. Add tsp amounts of cold water...
                </div>
            `;

            // Populate Quarantine Zone
            colQuarantine.innerHTML = `
                <h3>Quarantine Zone</h3>
                <div class="node node-danger">
                    <span class="badge badge-danger">Fatal: Orphaned Text</span><br>
                    "Blah blah about plates and silver ware. Ice cream or whip topping options"
                </div>
                <div class="node node-danger">
                    <span class="badge badge-danger">Fatal: Unmapped Node</span><br>
                    <strong>6.2.2 Study Inactivation:</strong> Study Inactivation is the process where Medidata inactivates the study...
                </div>
            `;

            // Show Download Button
            downloadBtn.style.display = 'block';
        }, 800);
    }
</script>
