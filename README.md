<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>System Workflow - Interactive Map</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }
        
        body {
            font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, sans-serif;
            background: linear-gradient(135deg, #0f2027 0%, #203a43 50%, #2c5364 100%);
            min-height: 100vh;
            padding: 20px;
            overflow-x: hidden;
        }
        
        .header {
            text-align: center;
            color: white;
            margin-bottom: 30px;
            animation: fadeIn 1s ease-in;
        }
        
        .header h1 {
            font-size: 3.5em;
            margin-bottom: 10px;
            text-shadow: 3px 3px 6px rgba(0,0,0,0.4);
        }
        
        .header p {
            font-size: 1.3em;
            opacity: 0.95;
        }
        
        .controls {
            text-align: center;
            margin-bottom: 30px;
        }
        
        .demo-button {
            background: linear-gradient(135deg, #e74c3c 0%, #c0392b 100%);
            color: white;
            border: none;
            padding: 15px 40px;
            border-radius: 30px;
            font-size: 1.2em;
            font-weight: 700;
            cursor: pointer;
            box-shadow: 0 10px 30px rgba(231, 76, 60, 0.4);
            transition: all 0.3s ease;
            margin: 0 10px;
        }
        
        .demo-button:hover {
            transform: translateY(-3px);
            box-shadow: 0 15px 40px rgba(231, 76, 60, 0.6);
        }
        
        .demo-button.reset {
            background: linear-gradient(135deg, #27ae60 0%, #229954 100%);
            box-shadow: 0 10px 30px rgba(39, 174, 96, 0.4);
        }
        
        .container {
            max-width: 1900px;
            margin: 0 auto;
        }
        
        .workflow-container {
            background: rgba(255, 255, 255, 0.98);
            border-radius: 30px;
            padding: 50px;
            box-shadow: 0 30px 90px rgba(0,0,0,0.5);
            position: relative;
        }
        
        .workflow-title {
            text-align: center;
            font-size: 2.5em;
            font-weight: 700;
            color: #2c3e50;
            margin-bottom: 20px;
        }
        
        .workflow-subtitle {
            text-align: center;
            font-size: 1.3em;
            color: #7f8c8d;
            margin-bottom: 50px;
        }
        
        svg {
            width: 100%;
            height: 900px;
            display: block;
        }
        
        /* Stage Labels */
        .stage-label {
            font-size: 22px;
            font-weight: 700;
            text-transform: uppercase;
            letter-spacing: 2px;
            fill: #2c3e50;
        }
        
        .stage-sublabel {
            font-size: 16px;
            fill: #7f8c8d;
            font-style: italic;
        }
        
        /* Node Styles */
        .node {
            cursor: pointer;
            transition: all 0.3s ease;
        }
        
        .node circle {
            transition: all 0.3s ease;
        }
        
        .node:hover circle {
            filter: drop-shadow(0 0 25px rgba(102, 126, 234, 0.9));
        }
        
        .node.inactive circle {
            opacity: 0.4;
            filter: grayscale(100%);
        }
        
        .node.inactive .status-badge {
            fill: #e74c3c;
        }
        
        .node.inactive .status-icon {
            fill: white;
        }
        
        /* Status Badge on Node */
        .status-badge {
            transition: all 0.3s ease;
        }
        
        /* Connection Lines */
        .connection {
            stroke: #95a5a6;
            stroke-width: 4;
            fill: none;
            transition: all 0.3s ease;
        }
        
        .connection.arrow {
            marker-end: url(#arrowhead);
        }
        
        .connection:hover {
            stroke: #667eea;
            stroke-width: 6;
            filter: drop-shadow(0 0 8px rgba(102, 126, 234, 0.6));
        }
        
        .connection.inactive {
            stroke: #e74c3c;
            stroke-dasharray: 10, 5;
            opacity: 0.5;
            animation: dashFlow 1s linear infinite;
        }
        
        @keyframes dashFlow {
            to { stroke-dashoffset: -15; }
        }
        
        .connection.active-flow {
            stroke: #27ae60;
            stroke-width: 6;
            animation: flowPath 2s ease-in-out infinite;
        }
        
        @keyframes flowPath {
            0%, 100% { 
                stroke: #27ae60;
                stroke-width: 6;
            }
            50% { 
                stroke: #2ecc71;
                stroke-width: 8;
            }
        }
        
        /* Info Panel */
        .info-panel {
            position: fixed;
            right: -550px;
            top: 50%;
            transform: translateY(-50%);
            width: 500px;
            max-height: 95vh;
            background: white;
            border-radius: 20px 0 0 20px;
            box-shadow: -10px 0 50px rgba(0,0,0,0.4);
            padding: 35px;
            transition: right 0.4s ease;
            overflow-y: auto;
            z-index: 1000;
        }
        
        .info-panel.active {
            right: 0;
        }
        
        .info-panel-header {
            display: flex;
            justify-content: space-between;
            align-items: flex-start;
            margin-bottom: 25px;
            padding-bottom: 20px;
            border-bottom: 3px solid #667eea;
        }
        
        .info-panel-icon {
            font-size: 3em;
            margin-right: 15px;
        }
        
        .info-panel-title-group {
            flex: 1;
        }
        
        .info-panel-title {
            font-size: 2em;
            font-weight: 700;
            color: #2c3e50;
            line-height: 1.2;
        }
        
        .info-panel-subtitle {
            font-size: 1.1em;
            color: #7f8c8d;
            font-style: italic;
            margin-top: 5px;
        }
        
        .close-btn {
            background: #e74c3c;
            color: white;
            border: none;
            width: 45px;
            height: 45px;
            border-radius: 50%;
            font-size: 1.8em;
            cursor: pointer;
            transition: all 0.3s ease;
            flex-shrink: 0;
        }
        
        .close-btn:hover {
            background: #c0392b;
            transform: rotate(90deg);
        }
        
        .info-section {
            margin-bottom: 30px;
        }
        
        .info-section-title {
            font-size: 1.5em;
            font-weight: 700;
            color: #2c3e50;
            margin-bottom: 15px;
            display: flex;
            align-items: center;
            gap: 10px;
        }
        
        .info-section-content {
            color: #555;
            line-height: 1.9;
            font-size: 1.08em;
        }
        
        .status-badge-large {
            display: inline-flex;
            align-items: center;
            gap: 10px;
            padding: 12px 25px;
            border-radius: 25px;
            font-weight: 700;
            font-size: 1.15em;
            margin: 15px 0;
        }
        
        .status-active-large {
            background: #d4edda;
            color: #155724;
            border: 2px solid #28a745;
        }
        
        .status-inactive-large {
            background: #f8d7da;
            color: #721c24;
            border: 2px solid #dc3545;
        }
        
        .example-box {
            background: linear-gradient(135deg, #f8f9fa 0%, #e9ecef 100%);
            padding: 25px;
            border-radius: 15px;
            margin-top: 15px;
            border-left: 5px solid #667eea;
            box-shadow: 0 5px 15px rgba(0,0,0,0.08);
        }
        
        .example-title {
            font-weight: 700;
            color: #667eea;
            margin-bottom: 12px;
            font-size: 1.2em;
        }
        
        .example-content {
            color: #2c3e50;
            line-height: 1.8;
        }
        
        .impact-box {
            background: linear-gradient(135deg, #e74c3c 0%, #c0392b 100%);
            color: white;
            padding: 25px;
            border-radius: 15px;
            margin-top: 15px;
            box-shadow: 0 10px 30px rgba(231, 76, 60, 0.3);
        }
        
        .impact-title {
            font-weight: 700;
            font-size: 1.3em;
            margin-bottom: 15px;
            display: flex;
            align-items: center;
            gap: 10px;
        }
        
        .impact-list {
            list-style: none;
            padding-left: 0;
        }
        
        .impact-list li {
            padding: 10px 0;
            border-bottom: 1px solid rgba(255,255,255,0.2);
            line-height: 1.6;
        }
        
        .impact-list li:last-child {
            border-bottom: none;
        }
        
        .connection-box {
            background: #fff3cd;
            padding: 20px;
            border-radius: 12px;
            margin-top: 15px;
            border-left: 5px solid #ffc107;
        }
        
        .connection-title {
            font-weight: 700;
            color: #856404;
            margin-bottom: 10px;
            font-size: 1.1em;
        }
        
        .legend-section {
            background: #e8f5e9;
            padding: 20px;
            border-radius: 12px;
            margin-top: 15px;
            border-left: 5px solid #4caf50;
        }
        
        .legend-title {
            font-weight: 700;
            color: #2e7d32;
            margin-bottom: 10px;
            font-size: 1.1em;
        }
        
        .metric-highlight {
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            color: white;
            padding: 20px;
            border-radius: 12px;
            margin-top: 15px;
            text-align: center;
        }
        
        .metric-number {
            font-size: 3em;
            font-weight: 700;
            margin-bottom: 10px;
        }
        
        .metric-label {
            font-size: 1.1em;
            opacity: 0.95;
        }
        
        /* Notification Banner */
        .notification {
            position: fixed;
            top: 100px;
            left: 50%;
            transform: translateX(-50%);
            background: linear-gradient(135deg, #e74c3c 0%, #c0392b 100%);
            color: white;
            padding: 25px 40px;
            border-radius: 15px;
            box-shadow: 0 15px 50px rgba(231, 76, 60, 0.5);
            z-index: 3000;
            min-width: 500px;
            max-width: 700px;
            opacity: 0;
            pointer-events: none;
            transition: all 0.4s ease;
        }
        
        .notification.show {
            opacity: 1;
            pointer-events: auto;
            animation: slideDown 0.4s ease-out;
        }
        
        @keyframes slideDown {
            from {
                transform: translateX(-50%) translateY(-30px);
                opacity: 0;
            }
            to {
                transform: translateX(-50%) translateY(0);
                opacity: 1;
            }
        }
        
        .notification-header {
            font-size: 1.6em;
            font-weight: 700;
            margin-bottom: 15px;
            display: flex;
            align-items: center;
            gap: 12px;
        }
        
        .notification-body {
            font-size: 1.1em;
            line-height: 1.7;
        }
        
        .notification-body strong {
            font-weight: 700;
            display: block;
            margin-top: 8px;
        }
        
        .notification.success {
            background: linear-gradient(135deg, #27ae60 0%, #229954 100%);
            box-shadow: 0 15px 50px rgba(39, 174, 96, 0.5);
        }
        
        /* Animations */
        @keyframes fadeIn {
            from { opacity: 0; transform: translateY(-20px); }
            to { opacity: 1; transform: translateY(0); }
        }
        
        @keyframes pulse {
            0%, 100% { transform: scale(1); }
            50% { transform: scale(1.05); }
        }
        
        /* Tooltip */
        .tooltip {
            position: absolute;
            background: rgba(0,0,0,0.95);
            color: white;
            padding: 12px 20px;
            border-radius: 10px;
            font-size: 1em;
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.3s ease;
            z-index: 2000;
            max-width: 300px;
            line-height: 1.6;
            box-shadow: 0 10px 30px rgba(0,0,0,0.5);
        }
        
        .tooltip.active {
            opacity: 1;
        }
        
        /* Responsive */
        @media (max-width: 1400px) {
            svg { height: 700px; }
            .info-panel { width: 450px; right: -500px; }
        }
        
        @media (max-width: 1024px) {
            .workflow-container { padding: 30px; }
            svg { height: 600px; }
            .info-panel {
                width: 100%;
                right: -100%;
                border-radius: 0;
                top: 0;
                transform: none;
                max-height: 100vh;
            }
        }
    </style>
</head>
<body>
    <div class="header">
        <h1>🗺️ Your System Workflow</h1>
        <p>Watch how data flows through 4 stages: Add → Link → Units → Pricing</p>
    </div>
    
    <div class="controls">
        <button class="demo-button" onclick="demonstrateDeactivation()">🚨 Demo: Turn OFF Product</button>
        <button class="demo-button reset" onclick="resetAll()">✅ Reset All to Active</button>
    </div>
    
    <div class="container">
        <div class="workflow-container">
            <div class="workflow-title">The Complete 4-Stage Workflow</div>
            <div class="workflow-subtitle">Add → Link → Units Created → Pricing Created • Click any circle to learn more</div>
            
            <svg viewBox="0 0 1800 900" xmlns="http://www.w3.org/2000/svg">
                <defs>
                    <!-- Arrow markers -->
                    <marker id="arrowhead" markerWidth="12" markerHeight="12" refX="11" refY="4" orient="auto">
                        <polygon points="0 0, 12 4, 0 8" fill="#95a5a6" />
                    </marker>
                    <marker id="arrowhead-active" markerWidth="12" markerHeight="12" refX="11" refY="4" orient="auto">
                        <polygon points="0 0, 12 4, 0 8" fill="#27ae60" />
                    </marker>
                    <marker id="arrowhead-inactive" markerWidth="12" markerHeight="12" refX="11" refY="4" orient="auto">
                        <polygon points="0 0, 12 4, 0 8" fill="#e74c3c" />
                    </marker>
                    
                    <!-- Gradients -->
                    <linearGradient id="code-gradient" x1="0%" y1="0%" x2="100%" y2="100%">
                        <stop offset="0%" style="stop-color:#667eea;stop-opacity:1" />
                        <stop offset="100%" style="stop-color:#764ba2;stop-opacity:1" />
                    </linearGradient>
                    
                    <linearGradient id="product-gradient" x1="0%" y1="0%" x2="100%" y2="100%">
                        <stop offset="0%" style="stop-color:#f093fb;stop-opacity:1" />
                        <stop offset="100%" style="stop-color:#f5576c;stop-opacity:1" />
                    </linearGradient>
                    
                    <linearGradient id="vendor-gradient" x1="0%" y1="0%" x2="100%" y2="100%">
                        <stop offset="0%" style="stop-color:#4facfe;stop-opacity:1" />
                        <stop offset="100%" style="stop-color:#00f2fe;stop-opacity:1" />
                    </linearGradient>
                    
                    <linearGradient id="link-gradient" x1="0%" y1="0%" x2="100%" y2="100%">
                        <stop offset="0%" style="stop-color:#ffa751;stop-opacity:1" />
                        <stop offset="100%" style="stop-color:#ffe259;stop-opacity:1" />
                    </linearGradient>
                    
                    <linearGradient id="unit-gradient" x1="0%" y1="0%" x2="100%" y2="100%">
                        <stop offset="0%" style="stop-color:#a8edea;stop-opacity:1" />
                        <stop offset="100%" style="stop-color:#fed6e3;stop-opacity:1" />
                    </linearGradient>
                    
                    <linearGradient id="scenario-gradient" x1="0%" y1="0%" x2="100%" y2="100%">
                        <stop offset="0%" style="stop-color:#96fbc4;stop-opacity:1" />
                        <stop offset="100%" style="stop-color:#f9f586;stop-opacity:1" />
                    </linearGradient>
                </defs>
                
                <!-- Stage Labels -->
                <text x="150" y="50" class="stage-label">STAGE 1</text>
                <text x="150" y="75" class="stage-sublabel">You Add Things</text>
                
                <text x="750" y="50" class="stage-label">STAGE 2</text>
                <text x="750" y="75" class="stage-sublabel">You Link Them</text>
                
                <text x="1250" y="50" class="stage-label">STAGE 3</text>
                <text x="1250" y="75" class="stage-sublabel">System Creates Units</text>
                
                <text x="1550" y="50" class="stage-label">STAGE 4</text>
                <text x="1550" y="75" class="stage-sublabel">System Creates Prices</text>
                
                <!-- Connection Lines - Properly positioned with curves -->
                
                <!-- Code to Product-Code Link -->
                <path class="connection arrow" d="M 285 280 Q 450 320, 635 385" data-from="code" data-to="product-code-link" id="conn-1"/>
                
                <!-- Product to Product-Code Link -->
                <path class="connection arrow" d="M 270 505 Q 450 450, 635 445" data-from="product" data-to="product-code-link" id="conn-2"/>
                
                <!-- Product-Code Link to Scenarios (STAGE 4 - longer arc) -->
                <path class="connection arrow" d="M 820 395 Q 1100 350, 1445 420" data-from="product-code-link" data-to="scenarios" id="conn-3"/>
                
                <!-- Product to Product-Vendor Link -->
                <path class="connection arrow" d="M 285 590 Q 450 620, 635 655" data-from="product" data-to="product-vendor-link" id="conn-4"/>
                
                <!-- Vendor to Product-Vendor Link -->
                <path class="connection arrow" d="M 285 720 Q 450 700, 635 690" data-from="vendor" data-to="product-vendor-link" id="conn-5"/>
                
                <!-- Product-Vendor Link to Unit (STAGE 3 - curved path) -->
                <path class="connection arrow" d="M 810 650 Q 1000 590, 1155 570" data-from="product-vendor-link" data-to="unit" id="conn-6"/>
                
                <!-- Unit to Scenarios (STAGE 3 to STAGE 4 - horizontal flow) -->
                <path class="connection arrow" d="M 1350 530 Q 1420 480, 1445 460" data-from="unit" data-to="scenarios" id="conn-7"/>
                
                <!-- STAGE 1: Foundation (What You Add) -->
                
                <!-- Billing Code Node -->
                <g class="node" data-id="code" onclick="showInfo('code')">
                    <circle cx="200" cy="250" r="95" fill="url(#code-gradient)" stroke="#fff" stroke-width="5"/>
                    
                    <!-- Icon -->
                    <text x="200" y="210" text-anchor="middle" fill="white" font-size="28" font-weight="700">📋</text>
                    
                    <!-- Main Label -->
                    <text x="200" y="245" text-anchor="middle" fill="white" font-size="20" font-weight="700">Billing Code</text>
                    
                    <!-- Example -->
                    <text x="200" y="270" text-anchor="middle" fill="white" font-size="14" font-weight="600">Ex: L1843</text>
                    <text x="200" y="290" text-anchor="middle" fill="white" font-size="13">Pays: $104.50</text>
                    
                    <!-- Badge -->
                    <rect x="145" y="305" width="110" height="28" rx="14" class="status-badge" fill="#27ae60"/>
                    <text x="200" y="323" text-anchor="middle" class="status-icon" fill="white" font-size="13" font-weight="700">✅ YOU ADD</text>
                </g>
                
                <!-- Product Node -->
                <g class="node" data-id="product" onclick="showInfo('product')">
                    <circle cx="200" cy="550" r="95" fill="url(#product-gradient)" stroke="#fff" stroke-width="5"/>
                    
                    <!-- Icon -->
                    <text x="200" y="510" text-anchor="middle" fill="white" font-size="28" font-weight="700">📦</text>
                    
                    <!-- Main Label -->
                    <text x="200" y="545" text-anchor="middle" fill="white" font-size="20" font-weight="700">Product</text>
                    
                    <!-- Example -->
                    <text x="200" y="570" text-anchor="middle" fill="white" font-size="14" font-weight="600">Ex: Knee Brace</text>
                    <text x="200" y="590" text-anchor="middle" fill="white" font-size="13">What you sell</text>
                    
                    <!-- Badge -->
                    <rect x="145" y="605" width="110" height="28" rx="14" class="status-badge" fill="#27ae60"/>
                    <text x="200" y="623" text-anchor="middle" class="status-icon" fill="white" font-size="13" font-weight="700">✅ YOU ADD</text>
                </g>
                
                <!-- Vendor Node -->
                <g class="node" data-id="vendor" onclick="showInfo('vendor')">
                    <circle cx="200" cy="750" r="95" fill="url(#vendor-gradient)" stroke="#fff" stroke-width="5"/>
                    
                    <!-- Icon -->
                    <text x="200" y="710" text-anchor="middle" fill="white" font-size="28" font-weight="700">🏢</text>
                    
                    <!-- Main Label -->
                    <text x="200" y="745" text-anchor="middle" fill="white" font-size="20" font-weight="700">Vendor</text>
                    
                    <!-- Example -->
                    <text x="200" y="770" text-anchor="middle" fill="white" font-size="14" font-weight="600">Ex: Vendor A</text>
                    <text x="200" y="790" text-anchor="middle" fill="white" font-size="13">Where you buy</text>
                    
                    <!-- Badge -->
                    <rect x="145" y="805" width="110" height="28" rx="14" class="status-badge" fill="#27ae60"/>
                    <text x="200" y="823" text-anchor="middle" class="status-icon" fill="white" font-size="13" font-weight="700">✅ YOU ADD</text>
                </g>
                
                <!-- STAGE 2: Connections (What You Link) -->
                
                <!-- Product-Code Link Node -->
                <g class="node" data-id="product-code-link" onclick="showInfo('product-code-link')">
                    <circle cx="725" cy="415" r="100" fill="url(#link-gradient)" stroke="#fff" stroke-width="5"/>
                    
                    <!-- Icon -->
                    <text x="725" y="370" text-anchor="middle" fill="white" font-size="26" font-weight="700">🔗</text>
                    
                    <!-- Main Label -->
                    <text x="725" y="400" text-anchor="middle" fill="white" font-size="18" font-weight="700">Billing Approval</text>
                    
                    <!-- Description -->
                    <text x="725" y="422" text-anchor="middle" fill="white" font-size="13" font-weight="600">Product ↔ Code</text>
                    <text x="725" y="440" text-anchor="middle" fill="white" font-size="12">"Can bill this way"</text>
                    
                    <!-- Badge -->
                    <rect x="670" y="455" width="110" height="28" rx="14" class="status-badge" fill="#f39c12"/>
                    <text x="725" y="473" text-anchor="middle" class="status-icon" fill="white" font-size="13" font-weight="700">🔗 YOU LINK</text>
                </g>
                
                <!-- Product-Vendor Link Node -->
                <g class="node" data-id="product-vendor-link" onclick="showInfo('product-vendor-link')">
                    <circle cx="725" cy="675" r="100" fill="url(#link-gradient)" stroke="#fff" stroke-width="5"/>
                    
                    <!-- Icon -->
                    <text x="725" y="630" text-anchor="middle" fill="white" font-size="26" font-weight="700">🔗</text>
                    
                    <!-- Main Label -->
                    <text x="725" y="660" text-anchor="middle" fill="white" font-size="18" font-weight="700">Supply Source</text>
                    
                    <!-- Description -->
                    <text x="725" y="682" text-anchor="middle" fill="white" font-size="13" font-weight="600">Product ↔ Vendor</text>
                    <text x="725" y="700" text-anchor="middle" fill="white" font-size="12">"Buy from them"</text>
                    
                    <!-- Badge -->
                    <rect x="670" y="715" width="110" height="28" rx="14" class="status-badge" fill="#f39c12"/>
                    <text x="725" y="733" text-anchor="middle" class="status-icon" fill="white" font-size="13" font-weight="700">🔗 YOU LINK</text>
                </g>
                
                <!-- STAGE 3: Units (What System Creates First) -->
                
                <!-- Unit Node -->
                <g class="node" data-id="unit" onclick="showInfo('unit')">
                    <circle cx="1250" cy="550" r="100" fill="url(#unit-gradient)" stroke="#fff" stroke-width="5"/>
                    
                    <!-- Icon -->
                    <text x="1250" y="505" text-anchor="middle" fill="#2c3e50" font-size="28" font-weight="700">🎯</text>
                    
                    <!-- Main Label -->
                    <text x="1250" y="535" text-anchor="middle" fill="#2c3e50" font-size="20" font-weight="700">Unit</text>
                    
                    <!-- Description -->
                    <text x="1250" y="558" text-anchor="middle" fill="#2c3e50" font-size="14" font-weight="600">Product + Vendor</text>
                    <text x="1250" y="576" text-anchor="middle" fill="#2c3e50" font-size="12">Cost: $45</text>
                    
                    <!-- Badge -->
                    <rect x="1175" y="590" width="150" height="28" rx="14" class="status-badge" fill="#9b59b6"/>
                    <text x="1250" y="608" text-anchor="middle" class="status-icon" fill="white" font-size="13" font-weight="700">⚙️ AUTO-CREATED</text>
                </g>
                
                <!-- STAGE 4: Pricing Options (What System Creates Second) -->
                
                <!-- Pricing Options Node -->
                <g class="node" data-id="scenarios" onclick="showInfo('scenarios')">
                    <circle cx="1550" cy="450" r="110" fill="url(#scenario-gradient)" stroke="#fff" stroke-width="5"/>
                    
                    <!-- Icon -->
                    <text x="1550" y="395" text-anchor="middle" fill="#2c3e50" font-size="32" font-weight="700">💰</text>
                    
                    <!-- Main Label -->
                    <text x="1550" y="430" text-anchor="middle" fill="#2c3e50" font-size="22" font-weight="700">Pricing</text>
                    <text x="1550" y="455" text-anchor="middle" fill="#2c3e50" font-size="22" font-weight="700">Options</text>
                    
                    <!-- Description -->
                    <text x="1550" y="480" text-anchor="middle" fill="#2c3e50" font-size="13" font-weight="600">Unit × Code</text>
                    <text x="1550" y="498" text-anchor="middle" fill="#2c3e50" font-size="12">Profit: $59.50</text>
                    
                    <!-- Badge -->
                    <rect x="1475" y="515" width="150" height="28" rx="14" class="status-badge" fill="#9b59b6"/>
                    <text x="1550" y="533" text-anchor="middle" class="status-icon" fill="white" font-size="13" font-weight="700">⚙️ AUTO-CREATED</text>
                </g>
            </svg>
        </div>
        
        <!-- Info Panel -->
        <div class="info-panel" id="infoPanel">
            <div class="info-panel-header">
                <div class="info-panel-icon" id="infoPanelIcon">📋</div>
                <div class="info-panel-title-group">
                    <div class="info-panel-title" id="infoPanelTitle">Select a node</div>
                    <div class="info-panel-subtitle" id="infoPanelSubtitle">Click any circle to learn more</div>
                </div>
                <button class="close-btn" onclick="closeInfo()">×</button>
            </div>
            <div id="infoPanelContent"></div>
        </div>
    </div>
    
    <div class="tooltip" id="tooltip"></div>
    
    <!-- Notification Banner -->
    <div class="notification" id="notification">
        <div class="notification-header" id="notificationHeader">
            🚨 Notification
        </div>
        <div class="notification-body" id="notificationBody">
            Message content here
        </div>
    </div>
    
    <script>
        // Notification system
        function showNotification(header, body, type = 'error') {
            const notification = document.getElementById('notification');
            const notificationHeader = document.getElementById('notificationHeader');
            const notificationBody = document.getElementById('notificationBody');
            
            notificationHeader.textContent = header;
            notificationBody.innerHTML = body;
            
            // Set type
            notification.className = 'notification show';
            if (type === 'success') {
                notification.classList.add('success');
            }
            
            // Auto-hide after 4 seconds
            setTimeout(() => {
                notification.classList.remove('show');
            }, 4000);
        }
        
        function demonstrateDeactivation() {
            // Deactivate product and show cascade effect
            const productNode = document.querySelector('[data-id="product"]');
            const productCodeLink = document.querySelector('[data-id="product-code-link"]');
            const productVendorLink = document.querySelector('[data-id="product-vendor-link"]');
            const unitNode = document.querySelector('[data-id="unit"]');
            const scenariosNode = document.querySelector('[data-id="scenarios"]');
            
            // Step 1: Deactivate product
            setTimeout(() => {
                productNode.classList.add('inactive');
                showNotification(
                    '🚨 STEP 1: Product Deactivated',
                    'The product "Knee Brace" is now <strong>INACTIVE ❌</strong><br><br>Watch what happens next...'
                );
            }, 100);
            
            // Step 2: Cascade to links
            setTimeout(() => {
                productCodeLink.classList.add('inactive');
                productVendorLink.classList.add('inactive');
                document.getElementById('conn-2').classList.add('inactive');
                document.getElementById('conn-4').classList.add('inactive');
                showNotification(
                    '🔗 STEP 2: Links Broken',
                    '• Billing Approval → <strong>OFF ❌</strong><br>• Supply Source → <strong>OFF ❌</strong><br><br>Cannot bill or purchase this product anymore!'
                );
            }, 4500);
            
            // Step 3: Cascade to unit
            setTimeout(() => {
                unitNode.classList.add('inactive');
                document.getElementById('conn-6').classList.add('inactive');
                showNotification(
                    '🎯 STEP 3: Unit Deactivated',
                    'Unit "Knee Brace - Vendor A" → <strong>OFF ❌</strong><br><br>This specific product-vendor combo is now unavailable!'
                );
            }, 9000);
            
            // Step 4: Cascade to scenarios
            setTimeout(() => {
                scenariosNode.classList.add('inactive');
                document.getElementById('conn-3').classList.add('inactive');
                document.getElementById('conn-7').classList.add('inactive');
                showNotification(
                    '💰 STEP 4: Pricing Options Gone',
                    '<strong>ALL Pricing Options → OFF ❌</strong><br><br>• 20-100+ scenarios affected!<br>• Sales team MUST STOP selling immediately!<br><br>Click "Reset All" to restore the system.'
                );
            }, 13500);
        }
        
        function resetAll() {
            document.querySelectorAll('.node').forEach(node => {
                node.classList.remove('inactive');
            });
            document.querySelectorAll('.connection').forEach(conn => {
                conn.classList.remove('inactive');
            });
            showNotification(
                '✅ System Reset Complete!',
                'Everything is back to <strong>ACTIVE ✅</strong> status.<br><br>Your system is fully operational again!',
                'success'
            );
        }
        
        const nodeData = {
            'code': {
                icon: '📋',
                title: 'Billing Code',
                subtitle: 'Insurance billing codes',
                category: 'YOU ADD THIS',
                description: 'Think of billing codes as the "language" you use to talk to insurance companies. Each code represents a specific type of medical equipment. When you submit a claim, the insurance company looks at the code and says "okay, for that code, we pay $X." Without the right code, you can\'t get paid by insurance!',
                bigPicture: {
                    title: '🌍 The Big Picture',
                    content: 'Insurance companies don\'t speak "Knee Brace" or "Wheelchair" - they speak in codes like L1843 or E1130. These codes are standardized across the entire healthcare industry. Medicare creates these codes, and everyone else follows. You MUST use the correct code to bill insurance, or your claim gets denied.'
                },
                howItWorks: {
                    title: '⚙️ How It Works',
                    content: `<strong>Step 1:</strong> Medicare publishes official codes (called HCPCS codes)<br>
                    <strong>Step 2:</strong> Each code has a description and reimbursement amount<br>
                    <strong>Step 3:</strong> You add the codes you use to your system<br>
                    <strong>Step 4:</strong> You link codes to products that qualify<br>
                    <strong>Step 5:</strong> When you sell, you bill insurance using that code<br>
                    <strong>Step 6:</strong> Insurance pays you the amount for that code<br><br>
                    <strong>Example:</strong> You sell a knee brace → You bill code L1843 → Insurance pays you $104.50`
                },
                whyMatters: {
                    title: '💡 Why This Matters',
                    content: 'Billing codes determine HOW MUCH MONEY you get paid! Same product can have different codes that pay different amounts. Choosing the right code can mean the difference between a 100% margin and a 200% margin. That\'s why the system tracks all available codes for each product!'
                },
                example: {
                    title: '📋 Real Example',
                    content: `<strong>Code:</strong> L1843<br>
                    <strong>Official Name:</strong> "Knee orthosis, single upright, thigh and calf, with adjustable flexion and extension joint"<br>
                    <strong>In Plain English:</strong> A knee brace with one metal bar that goes up your thigh and down your calf<br><br>
                    <strong>Reimbursement Rates:</strong><br>
                    💵 Medicare Pays: $104.50<br>
                    💵 Medicaid Pays: $95.00<br>
                    💵 Private Insurance: ~$115.00<br><br>
                    <strong>Used For:</strong> Post-injury recovery, ACL support, arthritis patients`
                },
                commonQuestions: {
                    title: '❓ Common Questions',
                    content: `<strong>Q: Can I make up my own codes?</strong><br>
                    A: No! You must use official Medicare/insurance codes only.<br><br>
                    <strong>Q: Can one product have multiple codes?</strong><br>
                    A: Yes! That's the smart strategy. Same product, different codes, different profits.<br><br>
                    <strong>Q: What if the code gets discontinued?</strong><br>
                    A: You must stop using it and find a replacement code, or you can't bill insurance.<br><br>
                    <strong>Q: Who decides what codes pay?</strong><br>
                    A: Medicare sets the rates. Private insurance usually follows similar amounts.`
                },
                legend: {
                    title: '📖 What This Means',
                    content: 'A billing code is your "product SKU" for the insurance world. Just like Amazon has ASINs, healthcare has HCPCS codes. You must speak their language to get paid. Simple as that!'
                },
                impact: {
                    title: '⚠️ What Happens if You Turn This OFF?',
                    items: [
                        '❌ Cannot create new "Billing Approvals" with this code',
                        '❌ ALL existing Billing Approvals using this code turn OFF',
                        '❌ ALL Pricing Options using this code turn OFF',
                        '📊 Could affect 10-50+ pricing options',
                        '⚠️ Products using ONLY this code cannot be billed at all',
                        '💡 You\'ll need to find a replacement code',
                        '📧 Sales team must stop quoting with this code',
                        '⏰ Usually happens when Medicare discontinues a code'
                    ]
                },
                connections: [
                    '➡️ You manually link this to Products to create "Billing Approvals"',
                    '➡️ Used by Pricing Options to calculate insurance reimbursement',
                    '➡️ Determines how much money you receive from insurance'
                ]
            },
            'product': {
                icon: '📦',
                title: 'Product',
                subtitle: 'What you sell to customers',
                category: 'YOU ADD THIS',
                description: 'Your actual medical equipment inventory. These are the physical items your customers receive. Each product can be billed under multiple codes and purchased from multiple vendors.',
                example: {
                    title: '📦 Real Example',
                    content: `<strong>Product:</strong> Knee Brace Standard<br>
                    <strong>Category:</strong> Orthotic Devices - Knee<br>
                    <strong>Description:</strong> Standard knee support brace with single upright, adjustable straps<br>
                    <strong>Used For:</strong> Post-injury recovery, arthritis support<br>
                    <strong>Can be billed as:</strong> L1843, L1851<br>
                    <strong>Available from:</strong> Vendor A ($45), Vendor B ($42)`
                },
                legend: {
                    title: '📖 What This Means',
                    content: 'A product is the item in your inventory that you sell and deliver to customers. One product can have multiple ways to bill it (different codes) and multiple suppliers (different vendors). This flexibility maximizes your profit opportunities.'
                },
                impact: {
                    title: '🚨 CRITICAL: What Happens if You Turn This OFF?',
                    items: [
                        '❌ ALL Billing Approvals for this product turn OFF',
                        '❌ ALL Supply Sources for this product turn OFF',
                        '❌ ALL Units of this product turn OFF',
                        '❌ ALL Pricing Options for this product turn OFF',
                        '🚨 MASSIVE IMPACT: Could affect 20-100+ pricing options!',
                        '🚫 Sales team MUST STOP selling this product immediately',
                        '📧 Customer notifications required',
                        '💡 Need replacement product or reactivate'
                    ]
                },
                connections: [
                    'You link this to Codes to create "Billing Approvals"',
                    'You link this to Vendors to create "Supply Sources"',
                    'Central hub that connects billing and supply chains'
                ]
            },
            'vendor': {
                icon: '🏢',
                title: 'Vendor',
                subtitle: 'Where you buy products from',
                category: 'YOU ADD THIS',
                description: 'Your suppliers who provide medical equipment. Each vendor has their own pricing, quality ratings, delivery terms, and product availability. You can have multiple vendors for the same product to ensure supply chain stability.',
                example: {
                    title: '🏢 Real Example',
                    content: `<strong>Vendor:</strong> Vendor A Medical Supply<br>
                    <strong>Contact:</strong> John Smith, (555) 123-4567<br>
                    <strong>Rating:</strong> 4 out of 5 stars<br>
                    <strong>Payment Terms:</strong> Net 30<br>
                    <strong>Shipping:</strong> Free over $500<br>
                    <strong>Lead Time:</strong> 14 days average<br>
                    <strong>Products:</strong> Knee braces, ankle braces, back braces<br>
                    <strong>Pricing:</strong> Knee Brace = $45 each`
                },
                legend: {
                    title: '📖 What This Means',
                    content: 'A vendor is your supplier relationship. They provide products at specific costs. Having multiple vendors for the same product gives you pricing flexibility, reduces supply risk, and creates negotiating leverage.'
                },
                impact: {
                    title: '🚨 MASSIVE IMPACT: What Happens if You Turn This OFF?',
                    items: [
                        '❌ ALL Supply Sources with this vendor turn OFF',
                        '❌ ALL Units from this vendor turn OFF',
                        '❌ ALL Pricing Options using this vendor turn OFF',
                        '🚨 ENORMOUS IMPACT: Could affect 100-500+ pricing options!',
                        '⚠️ Products with ONLY this vendor cannot be purchased',
                        '📦 Must find replacement vendors immediately',
                        '💰 Revenue impact could be $10K-50K+ per month',
                        '⏰ Allow 30 days for full vendor transition'
                    ]
                },
                connections: [
                    'You link this to Products to create "Supply Sources"',
                    'Determines costs used in all profit calculations',
                    'Critical for supply chain management'
                ]
            },
            'product-code-link': {
                icon: '🔗',
                title: 'Billing Approval',
                subtitle: 'Product approved for billing code',
                category: 'YOU CREATE THIS',
                description: 'This connection says "this product can be billed under this insurance code." You create this manually after verifying that the product meets the code\'s requirements and insurance will accept claims.',
                example: {
                    title: '🔗 Real Example',
                    content: `<strong>Link Created:</strong> Knee Brace Standard → Code L1843<br><br>
                    <strong>What this means:</strong><br>
                    "We can bill insurance companies using code L1843 when we sell this knee brace."<br><br>
                    <strong>Why you create this:</strong><br>
                    • Product meets L1843 specifications<br>
                    • Insurance companies accept this billing<br>
                    • Creates pricing options automatically<br><br>
                    <strong>Result:</strong> System creates pricing options for all vendor combinations of this product using this code`
                },
                legend: {
                    title: '📖 What This Means',
                    content: 'A billing link tells your system "yes, we can legally and properly bill this product under this code." Without this link, you cannot bill insurance for that product-code combination, even if both exist in your system.'
                },
                impact: {
                    title: '⚠️ What Happens if You Turn This OFF?',
                    items: [
                        '❌ Cannot bill this product under this code anymore',
                        '❌ ALL Pricing Options for this product-code combo turn OFF',
                        '📊 Affects 4-20 pricing options (one per vendor)',
                        '⚠️ If this was product\'s ONLY billing code, product cannot be billed!',
                        '💡 Need alternative code or reactivate this link',
                        '📧 Notify sales team which scenarios are no longer valid'
                    ]
                },
                connections: [
                    'Links: Billing Code + Product',
                    'Triggers: Pricing Options creation for all vendors',
                    'Determines: Which reimbursement rate applies'
                ]
            },
            'product-vendor-link': {
                icon: '🔗',
                title: 'Supply Source',
                subtitle: 'Vendor supplies this product',
                category: 'YOU CREATE THIS',
                description: 'This connection says "we buy this product from this vendor at this cost." You create this manually and enter the negotiated cost per unit. This triggers automatic Unit creation.',
                example: {
                    title: '🔗 Real Example',
                    content: `<strong>Link Created:</strong> Knee Brace Standard ← Vendor A<br><br>
                    <strong>Details you enter:</strong><br>
                    • Cost: $45 per unit<br>
                    • MOQ: 50 units minimum<br>
                    • Lead Time: 14 days<br>
                    • Shipping: Included<br><br>
                    <strong>What happens automatically:</strong><br>
                    ⚙️ System creates Unit "Knee Brace - Vendor A"<br>
                    ⚙️ System creates pricing options for all billing codes<br>
                    ⚙️ System calculates profit margins automatically<br><br>
                    <strong>Result:</strong> Ready to sell at optimal profit!`
                },
                legend: {
                    title: '📖 What This Means',
                    content: 'A supply link connects your product catalog to your supplier relationships. It establishes cost, which is critical for calculating profit margins. This link triggers the entire automation chain that creates your pricing options.'
                },
                impact: {
                    title: '⚠️ What Happens if You Turn This OFF?',
                    items: [
                        '❌ Cannot purchase this product from this vendor',
                        '❌ Unit for this product-vendor combo turns OFF',
                        '❌ ALL Pricing Options using this unit turn OFF',
                        '📊 Affects 4-10 pricing options (one per billing code)',
                        '⚠️ If this was product\'s ONLY vendor, product cannot be purchased!',
                        '💡 Need alternative vendor or reactivate',
                        '✅ Other vendors for same product still work'
                    ]
                },
                connections: [
                    'Links: Product + Vendor',
                    'Triggers: Unit auto-creation',
                    'Determines: Cost used in all profit calculations',
                    'Enables: Pricing Options for this specific supplier'
                ]
            },
            'unit': {
                icon: '🎯',
                title: 'Unit',
                subtitle: 'Specific product from specific vendor',
                category: 'AUTO-CREATED BY SYSTEM',
                description: 'The system automatically creates a Unit when you make a Supply Source. A Unit represents one specific product from one specific vendor at a specific cost. It\'s the building block for pricing options.',
                example: {
                    title: '🎯 Real Example',
                    content: `<strong>Unit Created:</strong> "Knee Brace Standard - Vendor A"<br><br>
                    <strong>Created automatically from:</strong><br>
                    • Product: Knee Brace Standard<br>
                    • Vendor: Vendor A<br>
                    • Cost: $45<br><br>
                    <strong>What this represents:</strong><br>
                    "A knee brace that costs us $45 when we buy it from Vendor A"<br><br>
                    <strong>What happens next:</strong><br>
                    ⚙️ For each billing code (L1843, L1851)...<br>
                    ⚙️ System creates a pricing option<br>
                    ⚙️ Calculates: Insurance Pay - $45 Cost = Profit<br><br>
                    <strong>If you have 3 vendors:</strong> 3 units created<br>
                    <strong>If you have 2 codes:</strong> 3 units × 2 codes = 6 pricing options!`
                },
                legend: {
                    title: '📖 What This Means',
                    content: 'A Unit is like a "SKU with cost attached." Same product from different vendors = different units with different costs = different profit margins. The system uses units to automatically calculate every possible pricing scenario.'
                },
                impact: {
                    title: '⚠️ What Happens if You Turn This OFF?',
                    items: [
                        '❌ ALL Pricing Options using this unit turn OFF',
                        '📊 Affects 4-10 pricing options (one per billing code)',
                        '✅ Other vendors\' units still work (isolated impact)',
                        'ℹ️ Usually turns OFF automatically when Supply Source turns OFF',
                        '💡 To fix: Reactivate the Supply Source'
                    ]
                },
                connections: [
                    'Created by: Supply Source (automatic)',
                    'Combined with: Billing Approvals',
                    'Creates: Pricing Options (one per billing code)',
                    'Provides: Cost for profit calculations'
                ]
            },
            'scenarios': {
                icon: '💰',
                title: 'Pricing Options',
                subtitle: 'Ready-to-quote pricing scenarios',
                category: 'AUTO-CREATED BY SYSTEM',
                description: 'The final output! System automatically creates a pricing option for every Unit × Billing Code combination. Each option shows you exactly what to charge, what insurance pays, and your profit. Sales team picks the best option for each customer.',
                example: {
                    title: '💰 Real Example',
                    content: `<strong>Pricing Option:</strong> "Vendor A - L1843"<br><br>
                    <strong>Created automatically from:</strong><br>
                    • Product: Knee Brace Standard<br>
                    • Vendor: Vendor A<br>
                    • Unit: Knee Brace - Vendor A<br>
                    • Billing Code: L1843<br><br>
                    <strong>The Numbers:</strong><br>
                    💵 Your Cost: $45.00<br>
                    💰 Insurance Pays: $104.50<br>
                    ✅ Your Profit: $59.50<br>
                    📊 Margin: 132%<br><br>
                    <strong>How many get created?</strong><br>
                    • 2 vendors × 2 billing codes = 4 pricing options<br>
                    • 3 vendors × 3 billing codes = 9 pricing options<br>
                    • All created automatically in seconds!<br><br>
                    <strong>Sales team uses this to:</strong><br>
                    • Quote customers the right price<br>
                    • Choose most profitable option<br>
                    • See all alternatives at a glance`
                },
                legend: {
                    title: '📖 What This Means',
                    content: 'Pricing Options are your "menu" for selling products. Each option is a specific way to sell a specific product: which vendor to buy from, which code to bill with, and exactly how much profit you make. Your system maintains all options automatically!'
                },
                impact: {
                    title: '✨ The Magic Formula',
                    items: [
                        '📐 Formula: Units × Billing Codes = Pricing Options',
                        '⚙️ Example: 2 vendors × 2 codes = 4 options',
                        '⚙️ Example: 3 vendors × 3 codes = 9 options',
                        '⚙️ Example: 5 vendors × 4 codes = 20 options',
                        '⚡ ALL created automatically!',
                        '💰 ALL profit margins calculated instantly!',
                        '🎯 Sales team sees best margins first',
                        '📊 Update cost once, all options recalculate',
                        '✅ Always accurate, always current'
                    ]
                },
                connections: [
                    'Created from: Unit + Billing Approval',
                    'Shows: Complete pricing with profit margins',
                    'Used by: Sales team for customer quotes',
                    'Updated: Automatically when costs or rates change'
                ]
            }
        };
        
        function showInfo(nodeId) {
            const data = nodeData[nodeId];
            const panel = document.getElementById('infoPanel');
            const icon = document.getElementById('infoPanelIcon');
            const title = document.getElementById('infoPanelTitle');
            const subtitle = document.getElementById('infoPanelSubtitle');
            const content = document.getElementById('infoPanelContent');
            
            icon.textContent = data.icon;
            title.textContent = data.title;
            subtitle.textContent = data.subtitle;
            
            let html = `
                <div class="info-section">
                    <div class="status-badge-large status-active-large">
                        <span>✅</span>
                        <span>${data.category}</span>
                    </div>
                </div>
                
                <div class="info-section">
                    <div class="info-section-title">📝 What This Is</div>
                    <div class="info-section-content">${data.description}</div>
                </div>
            `;
            
            if (data.bigPicture) {
                html += `
                    <div class="info-section">
                        <div class="legend-section">
                            <div class="legend-title">${data.bigPicture.title}</div>
                            <div>${data.bigPicture.content}</div>
                        </div>
                    </div>
                `;
            }
            
            if (data.howItWorks) {
                html += `
                    <div class="info-section">
                        <div class="connection-box">
                            <div class="connection-title">${data.howItWorks.title}</div>
                            <div>${data.howItWorks.content}</div>
                        </div>
                    </div>
                `;
            }
            
            if (data.whyMatters) {
                html += `
                    <div class="info-section">
                        <div class="metric-highlight">
                            <div style="font-size: 1.2em; font-weight: 700; margin-bottom: 10px;">${data.whyMatters.title}</div>
                            <div style="font-size: 1em; line-height: 1.7;">${data.whyMatters.content}</div>
                        </div>
                    </div>
                `;
            }
            
            html += `
                <div class="info-section">
                    <div class="example-box">
                        <div class="example-title">${data.example.title}</div>
                        <div class="example-content">${data.example.content}</div>
                    </div>
                </div>
            `;
            
            if (data.commonQuestions) {
                html += `
                    <div class="info-section">
                        <div class="example-box" style="border-left-color: #3498db;">
                            <div class="example-title" style="color: #3498db;">${data.commonQuestions.title}</div>
                            <div class="example-content">${data.commonQuestions.content}</div>
                        </div>
                    </div>
                `;
            }
            
            if (data.legend) {
                html += `
                    <div class="info-section">
                        <div class="legend-section">
                            <div class="legend-title">${data.legend.title}</div>
                            <div>${data.legend.content}</div>
                        </div>
                    </div>
                `;
            }
            
            if (data.impact) {
                html += `
                    <div class="info-section">
                        <div class="impact-box">
                            <div class="impact-title">${data.impact.title}</div>
                            <ul class="impact-list">
                                ${data.impact.items.map(item => `<li>${item}</li>`).join('')}
                            </ul>
                        </div>
                    </div>
                `;
            }
            
            if (data.connections) {
                html += `
                    <div class="info-section">
                        <div class="connection-box">
                            <div class="connection-title">🔗 Connections & Flow</div>
                            ${data.connections.map(conn => `<div style="padding: 5px 0;">${conn}</div>`).join('')}
                        </div>
                    </div>
                `;
            }
            
            content.innerHTML = html;
            panel.classList.add('active');
        }
        
        function closeInfo() {
            document.getElementById('infoPanel').classList.remove('active');
        }
        
        // Hover effects
        document.querySelectorAll('.node').forEach(node => {
            node.addEventListener('mouseenter', function() {
                const nodeId = this.getAttribute('data-id');
                
                // Highlight connected paths
                document.querySelectorAll(`.connection[data-from="${nodeId}"], .connection[data-to="${nodeId}"]`).forEach(path => {
                    path.classList.add('active-flow');
                });
            });
            
            node.addEventListener('mouseleave', function() {
                document.querySelectorAll('.connection').forEach(path => {
                    path.classList.remove('active-flow');
                });
            });
        });
        
        // Connection tooltips
        const tooltip = document.getElementById('tooltip');
        document.querySelectorAll('.connection').forEach(conn => {
            conn.addEventListener('mouseenter', function(e) {
                const from = this.getAttribute('data-from');
                const to = this.getAttribute('data-to');
                
                const fromTitle = nodeData[from]?.title || from;
                const toTitle = nodeData[to]?.title || to;
                
                tooltip.textContent = `${fromTitle} → ${toTitle}`;
                tooltip.style.left = e.pageX + 15 + 'px';
                tooltip.style.top = e.pageY + 15 + 'px';
                tooltip.classList.add('active');
            });
            
            conn.addEventListener('mouseleave', function() {
                tooltip.classList.remove('active');
            });
            
            conn.addEventListener('mousemove', function(e) {
                tooltip.style.left = e.pageX + 15 + 'px';
                tooltip.style.top = e.pageY + 15 + 'px';
            });
        });
        
        // Removed auto-show on load - user can click any node to explore
    </script>
</body>
</html>
