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

    /* Collapsible Tree Architecture */
    details.node-tree { margin-bottom: 8px; background: #fff; border-radius: 6px; overflow: hidden; }
    details.node-tree > summary {
        padding: 10px 12px; font-weight: bold; cursor: pointer; 
        list-style: none; display: flex; align-items: center; font-size: 0.95rem;
    }
    details.node-tree > summary::-webkit-details-marker { display: none; }
    details.node-tree > summary::before {
        content: "▶"; font-size: 0.75rem; margin-right: 10px; color: #6b7280; transition: transform 0.2s;
    }
    details[open].node-tree > summary::before { transform: rotate(90deg); }
    .tree-content { padding: 12px; border-top: 1px solid rgba(0,0,0,0.05); display: flex; flex-direction: column; gap: 8px; }

    /* Thematic Tree Styles */
    details.ghost-tree { border: 1px dashed #9ca3af; background: #f3f4f6; }
    details.ghost-tree > summary { color: #4b5563; }
    
    details.safe-tree { border: 1px solid #34d399; background: #ecfdf5; }
    details.safe-tree > summary { color: #065f46; }

    details.danger-tree { border: 1px solid #f87171; background: #fef2f2; }
    details.danger-tree > summary { color: #991b1b; }

    /* Leaf Node Styles */
    .node { padding: 10px; border-radius: 4px; font-size: 0.85rem; line-height: 1.4; background: white; }
    .node-ghost { border: 1px solid #d1d5db; color: #4b5563; }
    .node-safe { border: 1px solid #6ee7b7; color: #065f46; }
    .node-danger { border: 1px solid #fca5a5; color: #991b1b; }
    
    .badge { 
        font-size: 0.65rem; font-weight: bold; text-transform: uppercase; 
        display: inline-block; margin-bottom: 4px; padding: 2px 6px; border-radius: 4px;
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
            
            <details class="node-tree ghost-tree">
                <summary>Header Content</summary>
                <div class="tree-content">
                    <div class="node node-ghost"><strong>Purpose</strong><br>Standard Text</div>
                    <div class="node node-ghost"><strong>Scope</strong><br>Standard Text</div>
                </div>
            </details>

            <details class="node-tree ghost-tree">
                <summary>Core Requirements</summary>
                <div class="tree-content">
                    <div class="node node-ghost"><strong>Responsibilities</strong><br>Table: Role | Responsibility</div>
                    <div class="node node-ghost"><strong>Definitions</strong><br>Table: Term | Definition</div>
                </div>
            </details>

            <details class="node-tree ghost-tree">
                <summary>Execution</summary>
                <div class="tree-content">
                    <div class="node node-ghost"><strong>Procedure</strong><br>Table: Step | Role | Procedure</div>
                    <div class="node node-ghost"><strong>Study Inactivation</strong><br>Standard Text</div>
                </div>
            </details>

            <details class="node-tree ghost-tree">
                <summary>Administrative</summary>
                <div class="tree-content">
                    <div class="node node-ghost"><strong>References</strong><br>Standard Text</div>
                    <div class="node node-ghost"><strong>Documentation Requirements</strong><br>Standard Text</div>
                    <div class="node node-ghost"><strong>Appendices</strong><br>Standard Text</div>
                    <div class="node node-ghost"><strong>Document History</strong><br>Table: Ver. | Revision Type | Effective Date...</div>
                    <div class="node node-ghost"><strong>Update Summary</strong><br>Table</div>
                </div>
            </details>
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

    dropzone.addEventListener('click', () => fileInput.click());
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
        
        setTimeout(() => {
            dropzoneText.innerHTML = `<strong>Selected:</strong> ${fileName} <br><span style="color: #059669;">✔ Triage Complete</span>`;
            
            // Populate Conforming Trunk with Collapsible Branches
            colTrunk.innerHTML = `
                <h3>Conforming Trunk</h3>
                
                <details class="node-tree safe-tree" open>
                    <summary>Header Content</summary>
                    <div class="tree-content">
                        <div class="node node-safe">
                            <span class="badge badge-safe">Mapped: Purpose</span><br>
                            <strong>Standard Text:</strong> This Business Practice (BP) provides guidance baking an apple pie
                        </div>
                        <div class="node node-safe">
                            <span class="badge badge-safe">Mapped: Scope</span><br>
                            <strong>Standard Text:</strong> This BP applies to pie baking
                        </div>
                    </div>
                </details>

                <details class="node-tree safe-tree">
                    <summary>Core Requirements</summary>
                    <div class="tree-content">
                        <div class="node node-safe">
                            <span class="badge badge-safe">Mapped: Responsibilities</span><br>
                            <strong>Cook:</strong> Follow this BP when baking apple pies. Gathers ingredients and equipment as described in procedure.
                        </div>
                        <div class="node node-safe">
                            <span class="badge badge-safe">Mapped: Definitions</span><br>
                            <strong>Pie:</strong> A baked dish with a pastry crust and a filling, which can be sweet or savory.
                        </div>
                    </div>
                </details>

                <details class="node-tree safe-tree">
                    <summary>Execution</summary>
                    <div class="tree-content">
                        <div class="node node-safe">
                            <span class="badge badge-safe">Mapped: Procedure</span><br>
                            <strong>Cook:</strong> Mix dough until just coming together. Add tsp amounts of cold water until crumbly dough is achieved. Wrap in plastic wrap and rest refrigerated for a minimum of 1 hour.
                        </div>
                        <div class="node node-safe">
                            <span class="badge badge-safe">Mapped: Study Inactivation</span><br>
                            <strong>Standard Text:</strong> Study Inactivation is the process where Medidata inactivates the study in Rave and it is no longer visible to any users.
                        </div>
                    </div>
                </details>
                
                <details class="node-tree safe-tree">
                    <summary>Administrative</summary>
                    <div class="tree-content">
                        <div class="node node-safe">
                            <span class="badge badge-safe">Mapped: References</span><br>
                            <strong>Standard Text:</strong> JOC – Joy of Cooking<br>TFC- The French Chef – Julia Child
                        </div>
                    </div>
                </details>
            `;

            // Populate Quarantine Zone with Collapsible Branches
            colQuarantine.innerHTML = `
                <h3>Quarantine Zone</h3>
                
                <details class="node-tree danger-tree" open>
                    <summary>Fatal: Orphaned Text</summary>
                    <div class="tree-content">
                        <div class="node node-danger">
                            "Blah blah about plates and silver ware"
                        </div>
                        <div class="node node-danger">
                            "Ice cream or whip topping options"
                        </div>
                    </div>
                </details>

                <details class="node-tree danger-tree" open>
                    <summary>Fatal: Unmapped Nodes</summary>
                    <div class="tree-content">
                        <div class="node node-danger">
                            <strong>Appendix 1 – Cooks Notes:</strong> Adde bullet point cooking notes here: These could be silly asides only Sarah would get
                        </div>
                    </div>
                </details>
            `;

            downloadBtn.style.display = 'block';
        }, 800);
    }
</script>
