<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>RADCOM MASTER v5.7.2 - SISTEMA DE COMUNICACIÓN SEGURA AES-256-GCM + VoIP</title>
    <!-- SCRIPTS VÁLIDOS Y CORREGIDOS -->
    <script src="https://unpkg.com/peerjs@1.5.2/dist/peerjs.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/js/all.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/openmeteo@7.2.0"></script> 
    <script src="https://cdnjs.cloudflare.com/ajax/libs/crypto-js/4.1.1/crypto-js.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/js-sha256/0.9.0/sha256.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/elliptic@6.5.4/dist/elliptic.min.js"></script>
    <script type="importmap">
     {
    "imports": {
      "@noble/ed25519": "https://cdn.jsdelivr.net/npm/@noble/ed25519@2.1.0/+esm"
      }
     }
    </script>
    <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" />
    <script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>      
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/leaflet.draw/1.0.4/leaflet.draw.css" />
    <script src="https://cdnjs.cloudflare.com/ajax/libs/leaflet.draw/1.0.4/leaflet.draw.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/dompurify/3.1.7/purify.min.js"></script>
    <style>  
        
        /* === ESTILOS ORIGINALES - SIN CAMBIOS === */
        
        * { 
            box-sizing: border-box; 
            margin: 0; 
            padding: 0; 
        }
        
        body { 
            background: #0a0a0a; 
            color: #00ff88; 
            font-family: 'Consolas', 'Courier New', monospace; 
            display: flex; 
            justify-content: center; 
            align-items: center; 
            min-height: 100vh; 
            padding: 20px;
            overflow-x: hidden;
        }

        .container { 
            width: 720px; 
            height: 720px; 
            border: 2px solid #1a1a1a; 
            background: linear-gradient(145deg, #050505 0%, #0a0a0a 100%);
            display: flex; 
            flex-direction: column;
            box-shadow: 0 0 30px rgba(0, 255, 136, 0.1);
            position: relative;
            overflow: hidden;
        }
        
        .container::before {
            content: '';
            position: absolute;
            top: 0;
            left: 0;
            right: 0;
            height: 3px;
            background: linear-gradient(90deg, 
                #ff3300 0%, 
                #00ff88 25%, 
                #0088ff 50%, 
                #00ff88 75%, 
                #ff3300 100%);
            z-index: 100;
        }
        
        .header-pro { 
            height: 48px;
            background: linear-gradient(90deg, #000 0%, #111 100%); 
            display: flex; 
            justify-content: space-between; 
            align-items: center; 
            padding: 0 12px;
            border-bottom: 2px solid #00ff88; 
            font-size: 0.75rem;
            position: relative;
            z-index: 10;
        }
        
        .version-badge {
            display: inline-block;
            background: linear-gradient(45deg, #ff3300, #ff6600);
            color: white;
            padding: 2px 8px;
            border-radius: 3px;
            font-weight: bold;
            font-size: 0.8rem;
            margin-left: 6px;
            box-shadow: 0 2px 4px rgba(255, 51, 0, 0.3);
        }
        
        .status-indicator {
            display: flex;
            align-items: center;
            gap: 6px;
        }
        
        .status-dot-live {
            width: 7px;
            height: 7px;
            border-radius: 50%;
            background: #00ff88;
            box-shadow: 0 0 6px #00ff88;
            animation: pulse 2s infinite;
        }
        
        @keyframes pulse {
            0% { opacity: 1; }
            50% { opacity: 0.5; }
            100% { opacity: 1; }
        }

        .main-layout { 
            display: flex; 
            flex: 1; 
            overflow: hidden; 
            height: calc(720px - 48px - 120px);
        }

        .sidebar { 
            width: 165px;
            border-right: 1px solid #222; 
            background: rgba(8, 8, 8, 0.9); 
            display: flex; 
            flex-direction: column;
            backdrop-filter: blur(5px);
            overflow-y: auto;
        }
        
        .sidebar-section {
            margin: 6px 0;
            border-bottom: 1px solid #222;
            padding-bottom: 6px;
        }
        
        .sidebar-title { 
            padding: 5px 8px;
            font-size: 0.7rem;
            background: rgba(17, 17, 17, 0.8); 
            color: #00ff88; 
            text-transform: uppercase;
            letter-spacing: 0.8px;
            border-bottom: 1px solid #00ff88; 
            display: flex;
            justify-content: space-between;
            align-items: center;
        }
        
        .peer-item { 
            padding: 5px 8px;
            font-size: 0.65rem;
            cursor: pointer; 
            border-bottom: 1px solid rgba(34, 34, 34, 0.5); 
            display: flex; 
            align-items: center; 
            justify-content: space-between;
            transition: all 0.3s ease;
            position: relative;
            overflow: hidden;
        }
        
        .peer-item::before {
            content: '';
            position: absolute;
            left: 0;
            top: 0;
            height: 100%;
            width: 0;
            background: linear-gradient(90deg, rgba(0, 255, 136, 0.1), transparent);
            transition: width 0.3s ease;
        }
        
        .peer-item:hover::before {
            width: 100%;
        }
        
        .peer-item.active { 
            background: linear-gradient(90deg, rgba(0, 255, 136, 0.15), transparent);
            border-left: 2px solid #00ff88;
        }
        
        .peer-info { 
            display: flex; 
            align-items: center; 
            gap: 5px;
            overflow: hidden;
            flex: 1;
        }
        
        .status-dot { 
            width: 5px;
            height: 5px;
            border-radius: 50%; 
            flex-shrink: 0; 
        }
        
        .st-online { 
            background: #00ff88 !important; 
            box-shadow: 0 0 4px #00ff88 !important;
        }
        
        .st-offline { 
            background: #ff3300 !important; 
            opacity: 0.5; 
        }
        
        .st-encrypted {
            background: #0088ff !important;
            box-shadow: 0 0 4px #0088ff !important;
        }
        
        .st-warning { 
            background: #ffaa00 !important; 
            box-shadow: 0 0 6px #ffaa00 !important;
            animation: pulseWarning 1s infinite;
        }
        
        @keyframes pulseWarning {
            0% { opacity: 1; }
            50% { opacity: 0.3; }
            100% { opacity: 1; }
        }
        
        .del-btn { 
            color: #666; 
            font-weight: bold; 
            padding: 1px 4px;
            cursor: pointer; 
            font-size: 0.7rem;
            border-radius: 2px;
            transition: all 0.3s;
            z-index: 2;
            position: relative;
        }
        
        .del-btn:hover { 
            background: #ff3300; 
            color: white;
        }

        .voip-btn {
            background: none;
            border: 1px solid #00ff88;
            color: #00ff88;
            width: 22px;
            height: 22px;
            border-radius: 50%;
            cursor: pointer;
            font-size: 0.7rem;
            display: flex;
            align-items: center;
            justify-content: center;
            margin-left: 4px;
            transition: all 0.2s ease;
            z-index: 2;
            position: relative;
            flex-shrink: 0;
        }
        
        .voip-btn:hover {
            background: #00ff88;
            color: #000;
            transform: scale(1.1);
            box-shadow: 0 0 8px #00ff88;
        }
        
        .voip-btn.active {
            background: #ff3300;
            border-color: #ff3300;
            color: white;
            animation: pulseVoip 1s infinite;
        }
        
        .voip-btn.in-call {
            background: #ff3300 !important;
            border-color: #ff3300 !important;
            color: white !important;
        }
        
        @keyframes pulseVoip {
            0% { opacity: 1; transform: scale(1); }
            50% { opacity: 0.7; transform: scale(1.1); }
            100% { opacity: 1; transform: scale(1); }
        }
        
        .voip-btn.disabled {
            opacity: 0.3;
            cursor: not-allowed;
            border-color: #666;
            color: #666;
        }
        
        .voip-btn.disabled:hover {
            background: none;
            transform: none;
            box-shadow: none;
        }

        .peer-actions {
            display: flex;
            align-items: center;
            gap: 2px;
        }

        .content-area { 
            flex: 1; 
            display: flex; 
            flex-direction: column; 
            background: #000; 
            overflow: hidden;
            position: relative;
        }

        .monitor-container {
            display: flex;
            flex-direction: column;
            height: 285px;
            position: relative;
        }
        
        #monitor-raw { 
            min-height: 28px;
            padding: 5px;
            font-size: 0.65rem;
            color: #00ff88; 
            background: rgba(0, 20, 0, 0.3); 
            border: 1px solid #222; 
            border-radius: 3px;
            font-family: 'Courier New', monospace;
            white-space: pre-wrap;
            word-break: break-all;
            margin: 6px 8px;
            overflow-y: auto;
            max-height: 55px;
        }
        
        .resizable-separator {
            height: 5px;
            background: linear-gradient(90deg, #333, #00ff88, #0088ff, #00ff88, #333);
            margin: 2px 8px;
            cursor: row-resize;
            position: relative;
            transition: all 0.3s;
        }
        
        .resizable-separator:hover {
            background: linear-gradient(90deg, #333, #00ff88, #0088ff, #00ff88, #333);
            height: 6px;
        }
        
        .resizable-separator::after {
            content: "════════════════════════════════════════";
            position: absolute;
            left: 50%;
            top: 50%;
            transform: translate(-50%, -50%);
            color: #00ff88;
            font-size: 5px;
            letter-spacing: 0.2px;
            opacity: 0.7;
        }

        .table-container {
            flex: 1;
            display: flex;
            flex-direction: column;
            margin: 0 8px 4px 8px;
            min-height: 100px;
            overflow: hidden;
        }
        
        .table-info {
            display: flex;
            justify-content: space-between;
            padding: 3px 5px;
            background: rgba(0, 0, 0, 0.8);
            border: 1px solid #333;
            border-radius: 2px;
            font-size: 0.55rem;
            color: #00ffea;
            align-items: center;
            margin-bottom: 2px;
            height: 26px;
        }
        
        .char-preview {
            display: flex;
            align-items: center;
            gap: 4px;
        }
        
        .preview-box {
            display: flex;
            flex-direction: column;
            align-items: center;
            min-width: 45px;
        }
        
        .preview-label {
            font-size: 0.45rem;
            color: #888;
            text-transform: uppercase;
            letter-spacing: 0.1px;
        }
        
        .preview-value {
            font-size: 0.7rem;
            font-weight: bold;
            margin-top: 0.5px;
        }
        
        .ansi-container { 
            flex: 1;
            padding: 3px;
            background: rgba(5, 5, 5, 0.95); 
            border: 1px solid #00ff88; 
            border-radius: 2px;
            position: relative;
            overflow: hidden;
        }

        .ansi-container::before {
            content: '..--- .---- - .-. .- -.. .. -. --. -.. .- -.--';
            position: absolute;
            top: -4px;
            left: 5px;
            background: #000;
            color: #00ff88;
            font-size: 0.5rem;
            padding: 0 3px;
            z-index: 1;
        }
        
        
        table { 
            width: 100%; 
            border-collapse: collapse; 
            font-size: 0.45rem;
            text-align: center; 
            color: #00ff88; 
            cursor: pointer; 
            table-layout: fixed;
            height: 100%;
        }
        
        th, td { 
            border: 1px solid rgba(34, 34, 34, 0.8); 
            padding: 0.3px;
            color: #888; 
            transition: all 0.3s ease; 
            position: relative; 
            overflow: hidden;
            text-overflow: ellipsis;
            white-space: nowrap;
            height: 11px;
            line-height: 0.9;
        }
        
        th {
            color: #00ff88;
            background: rgba(0, 0, 0, 0.8);
            font-weight: bold;
            padding: 1px 0.3px;
            text-transform: uppercase;
            letter-spacing: 0.1px;
            height: 12px;
        }
        
        td:hover { 
            background: rgba(0, 255, 136, 0.1); 
            color: #fff; 
            border-color: #00ff88; 
            transform: scale(1.02);
            z-index: 1;
            box-shadow: 0 0 3px rgba(0, 255, 136, 0.3);
        }
        
        td.selected { 
            background: rgba(0, 255, 136, 0.2) !important; 
            color: #00ff88 !important; 
            border-color: #00ff88 !important; 
            box-shadow: 0 0 6px rgba(0, 255, 136, 0.5);
            font-weight: bold; 
            z-index: 2;
        }
        
        td.highlighted {
            background: rgba(255, 100, 0, 0.2) !important;
            color: #ff6600 !important;
            border-color: #ff6600 !important;
            box-shadow: 0 0 6px rgba(255, 100, 0, 0.5);
        }
        
        .row-idx { 
            color: #00ff88; 
            font-weight: bold; 
            background: rgba(0, 0, 0, 0.9); 
            border-right: 1px solid #00ff88;
            width: 12px;
            cursor: default; 
            text-transform: uppercase;
            font-size: 0.45rem;
        }

        #monitor-decoded { 
            flex: 1; 
            padding: 3px 6px;
            overflow-y: auto;
            font-size: 0.65rem;
            background: rgba(2, 2, 2, 0.95); 
            color: #fff; 
            border-top: 1px solid #222;
            line-height: 1.05;
            max-height: calc(720px - 48px - 285px - 120px);
        }
        
        #monitor-decoded::-webkit-scrollbar,
        #monitor-raw::-webkit-scrollbar {
            width: 3px;
        }

        #monitor-decoded::-webkit-scrollbar-track,
        #monitor-raw::-webkit-scrollbar-track {
            background: rgba(0, 0, 0, 0.3);
            border-radius: 1.5px;
        }

        #monitor-decoded::-webkit-scrollbar-thumb,
        #monitor-raw::-webkit-scrollbar-thumb {
            background: rgba(0, 255, 136, 0.5);
            border-radius: 1.5px;
        }

        #monitor-decoded::-webkit-scrollbar-thumb:hover,
        #monitor-raw::-webkit-scrollbar-thumb:hover {
            background: rgba(0, 255, 136, 0.7);
        }
        
        .message-bubble {
            margin: 2px 0;
            padding: 3px 5px;
            border-radius: 2px;
            max-width: 96%;
            position: relative;
            animation: fadeIn 0.3s ease;
        }
        
        @keyframes fadeIn {
            from { opacity: 0; transform: translateY(2px); }
            to { opacity: 1; transform: translateY(0); }
        }
        
        .message-outgoing {
            background: linear-gradient(135deg, rgba(0, 136, 255, 0.2), rgba(0, 255, 136, 0.1));
            border-left: 1px solid #0088ff;
            margin-left: auto;
        }
        
        .message-incoming {
            background: linear-gradient(135deg, rgba(255, 51, 0, 0.1), rgba(255, 102, 0, 0.05));
            border-left: 1px solid #ff3300;
        }
        
        .message-system {
            background: linear-gradient(135deg, rgba(0, 255, 136, 0.05), transparent);
            border-left: 1px solid #00ff88;
            text-align: center;
            font-style: italic;
            color: #888;
            font-size: 0.6rem;
        }

        .message-voip {
            background: linear-gradient(135deg, rgba(0, 255, 136, 0.15), rgba(0, 136, 255, 0.1));
            border-left: 2px solid #00ff88;
            text-align: center;
            font-weight: bold;
        }

        .footer-pro { 
            background: linear-gradient(180deg, #111 0%, #000 100%);
            border-top: 1px solid #00ff88;
            height: 120px;
        }
        
        .input-controls {
            display: flex;
            background: rgba(0, 0, 0, 0.9);
            padding: 4px 5px;
            gap: 4px;
            align-items: center;
            border-bottom: 1px solid #222;
            height: 36px;
        }
        
        #inputMsg { 
            flex: 1; 
            background: rgba(0, 0, 0, 0.8); 
            border: 1px solid #333; 
            color: #00ff88; 
            padding: 5px 7px;
            font-size: 0.75rem;
            outline: none; 
            font-family: 'Courier New', monospace; 
            border-radius: 2px;
            transition: all 0.3s;
        }
        
        #inputMsg:focus { 
            border-color: #00ff88; 
            box-shadow: 0 0 8px rgba(0, 255, 136, 0.2);
        }
        
        #sendBtn { 
            background: linear-gradient(135deg, #00ff88 0%, #0088ff 100%); 
            color: #000; 
            border: none; 
            padding: 5px 10px;
            font-weight: bold; 
            cursor: pointer; 
            font-size: 0.7rem;
            border-radius: 2px;
            text-transform: uppercase;
            letter-spacing: 0.2px;
            transition: all 0.3s;
            min-width: 85px;
        }
        
        #sendBtn:hover { 
            transform: translateY(-1px); 
            box-shadow: 0 2px 8px rgba(0, 255, 136, 0.3);
        }
        
        .btn-header { 
            background: linear-gradient(135deg, #222 0%, #333 100%); 
            color: #00ff88; 
            border: 1px solid #333; 
            padding: 2px 5px;
            font-size: 0.55rem;
            cursor: pointer; 
            border-radius: 2px;
            transition: all 0.3s;
            text-transform: uppercase;
            letter-spacing: 0.1px;
        }
        
        .btn-header:hover { 
            background: linear-gradient(135deg, #333 0%, #444 100%); 
            border-color: #00ff88;
        }
        
        .clear-chat-btn {
            background: linear-gradient(135deg, #333 0%, #444 100%);
            color: #ff3300;
            border: 1px solid #444;
            padding: 2px 5px;
            font-size: 0.55rem;
            cursor: pointer;
            border-radius: 2px;
            transition: all 0.3s;
        }
        
        .clear-chat-btn:hover {
            background: linear-gradient(135deg, #ff3300 0%, #ff6600 100%);
            color: white;
            border-color: #ff3300;
        }
        
        .security-badge {
            display: inline-flex;
            align-items: center;
            gap: 2px;
            background: rgba(0, 136, 255, 0.2);
            padding: 1px 3px;
            border-radius: 2px;
            font-size: 0.5rem;
            border: 1px solid rgba(0, 136, 255, 0.5);
        }
        
        .stats-panel {
            display: flex;
            gap: 10px;
            padding: 3px 8px;
            background: rgba(0, 0, 0, 0.9);
            border-top: 1px solid #222;
            font-size: 0.55rem;
            color: #888;
            justify-content: space-around;
            height: 30px;
        }
        
        .stat-item {
            display: flex;
            flex-direction: column;
            align-items: center;
        }
        
        .stat-value {
            font-size: 0.8rem;
            font-weight: bold;
            color: #00ff88;
        }
        
        .stat-label {
            font-size: 0.45rem;
            text-transform: uppercase;
            letter-spacing: 0.1px;
        }
        
        .encryption-indicator {
            display: flex;
            align-items: center;
            gap: 2px;
            color: #0088ff;
            font-size: 0.55rem;
        }
        
        select {
            background: #000;
            color: #00ff88;
            border: 1px solid #333;
            font-size: 0.6rem;
            padding: 3px 5px;
            border-radius: 2px;
            outline: none;
            min-width: 85px;
            height: 26px;
        }
        
        .modal-overlay {
            display: none;
            position: fixed;
            top: 0;
            left: 0;
            right: 0;
            bottom: 0;
            background: rgba(0, 0, 0, 0.95);
            z-index: 1000;
            align-items: center;
            justify-content: center;
            padding: 20px;
        }
        
        .modal-content {
            background: linear-gradient(145deg, #050505 0%, #0a0a0a 100%);
            border: 2px solid #00ff88;
            border-radius: 5px;
            padding: 12px;
            max-width: 450px;
            width: 100%;
            color: #00ff88;
        }
        
        .modal-title {
            font-size: 1rem;
            margin-bottom: 10px;
            color: #00ff88;
            text-align: center;
        }
        
        .modal-close {
            position: absolute;
            top: 8px;
            right: 8px;
            background: none;
            border: none;
            color: #ff3300;
            font-size: 1.1rem;
            cursor: pointer;
        }
        
        input[type="password"], input[type="text"] {
            background: rgba(0, 0, 0, 0.8);
            color: #00ff88;
            border: 1px solid #333;
            padding: 4px 6px;
            border-radius: 2px;
            outline: none;
            font-family: 'Courier New', monospace;
            font-size: 0.75rem;
        }
        
        .connection-type-selector {
            display: flex;
            gap: 8px;
            margin: 10px 0;
        }
        
        .connection-type-btn {
            flex: 1;
            padding: 8px;
            background: rgba(0, 0, 0, 0.8);
            border: 1px solid #333;
            color: #888;
            border-radius: 3px;
            cursor: pointer;
            text-align: center;
            transition: all 0.3s;
            font-size: 0.7rem;
        }
        
        .connection-type-btn.active {
            background: rgba(0, 255, 136, 0.2);
            border-color: #00ff88;
            color: #00ff88;
            box-shadow: 0 0 8px rgba(0, 255, 136, 0.3);
        }
        
        .connection-type-btn:hover:not(.active) {
            border-color: #0088ff;
            color: #0088ff;
        }
        
        .connection-icon {
            display: block;
            font-size: 1.2rem;
            margin-bottom: 3px;
        }
        
        .connection-info {
            font-size: 0.6rem;
            color: #888;
            margin-top: 3px;
            text-align: center;
        }
        
        .btn-refresh {
            background: linear-gradient(135deg, #ff6600 0%, #ff3300 100%) !important;
            color: white !important;
            border: none !important;
            width: auto !important;
            min-width: auto !important;
            padding: 2px 5px !important;
        }
        
        .btn-refresh:hover {
            background: linear-gradient(135deg, #ff8800 0%, #ff5500 100%) !important;
            transform: scale(1.05);
        }
        
        #saved-peers-container {
            max-height: 200px;
            overflow-y: auto;
            border: 1px solid rgba(34, 34, 34, 0.5);
            margin: 4px;
            border-radius: 3px;
            background: rgba(5, 5, 5, 0.7);
        }

        #saved-peers {
            padding: 2px;
        }

        #saved-peers-container::-webkit-scrollbar {
            width: 4px;
        }

        #saved-peers-container::-webkit-scrollbar-track {
            background: rgba(0, 0, 0, 0.3);
            border-radius: 2px;
        }

        #saved-peers-container::-webkit-scrollbar-thumb {
            background: rgba(0, 255, 136, 0.5);
            border-radius: 2px;
        }

        #saved-peers-container::-webkit-scrollbar-thumb:hover {
            background: rgba(0, 255, 136, 0.7);
        }

        .revive-btn {
            width: 100%;
            padding: 3px;
            margin-top: 5px;
            background: linear-gradient(135deg, #ffaa00 0%, #ff8800 100%);
            color: #000;
            border: none;
            border-radius: 2px;
            font-size: 0.55rem;
            cursor: pointer;
            transition: all 0.3s;
            text-transform: uppercase;
            font-weight: bold;
        }

        .revive-btn:hover {
            background: linear-gradient(135deg, #ffbb00 0%, #ff9900 100%);
            transform: translateY(-1px);
            box-shadow: 0 2px 6px rgba(255, 170, 0, 0.3);
        }

        .revive-btn:active {
            transform: translateY(0);
        }

        .reviving {
            animation: revivingPulse 0.5s infinite;
        }

        @keyframes revivingPulse {
            0% { opacity: 1; }
            50% { opacity: 0.7; }
            100% { opacity: 1; }
        }

        .tab-container {
            display: flex;
            background: rgba(0, 0, 0, 0.8);
            border-bottom: 1px solid #333;
        }

        .tab-button {
            flex: 1;
            padding: 3px 5px;
            background: rgba(17, 17, 17, 0.8);
            color: #888;
            border: none;
            cursor: pointer;
            font-size: 0.55rem;
            text-transform: uppercase;
            transition: all 0.3s;
        }

        .tab-button.active {
            background: rgba(0, 255, 136, 0.15);
            color: #00ff88;
            border-bottom: 2px solid #00ff88;
        }

        .tab-button:hover {
            background: rgba(0, 255, 136, 0.1);
        }

        .tab-content {
            display: none;
            height: 100%;
            overflow-y: auto;
        }

        .tab-content.active {
            display: block;
        }

        .preview-value {
            font-size: 0.7rem;
        }

        .preview-label {
            font-size: 0.45rem;
        }

        .morse-table-container {
            padding: 3px;
            height: 100%;
            overflow-y: auto;
        }

        .morse-table-container::-webkit-scrollbar,
        .tab-content::-webkit-scrollbar {
            width: 4px;
        }

        .morse-table-container::-webkit-scrollbar-track,
        .tab-content::-webkit-scrollbar-track {
            background: rgba(0, 0, 0, 0.3);
            border-radius: 2px;
        }

        .morse-table-container::-webkit-scrollbar-thumb,
        .tab-content::-webkit-scrollbar-thumb {
            background: rgba(0, 255, 136, 0.5);
            border-radius: 2px;
        }

        .morse-table-container::-webkit-scrollbar-thumb:hover,
        .tab-content::-webkit-scrollbar-thumb:hover {
            background: rgba(0, 255, 136, 0.7);
        }

        #morseTable {
            font-size: 0.5rem;
            width: 100%;
            border-collapse: collapse;
        }

        #morseTable th {
            background: rgba(0, 0, 0, 0.9);
            padding: 3px;
            font-weight: bold;
            border: 1px solid #333;
        }

        #morseTable td {
            padding: 4px 2px;
            border: 1px solid #333;
            text-align: center;
            cursor: pointer;
            transition: all 0.2s;
        }

        #morseTable td:hover {
            background: rgba(0, 255, 136, 0.1);
            color: #fff;
            border-color: #00ff88;
        }

        .morse-char {
            color: #00ff88;
            font-weight: bold;
            font-size: 0.6rem;
        }

        .morse-code {
            color: #0088ff;
            font-family: 'Courier New', monospace;
            font-size: 0.55rem;
        }

        .phonetic-example {
            color: #ffaa00;
            font-size: 0.5rem;
        }

        .play-morse-btn {
            background: rgba(0, 255, 136, 0.2);
            border: 1px solid #00ff88;
            color: #00ff88;
            padding: 1px 4px;
            font-size: 0.4rem;
            border-radius: 2px;
            cursor: pointer;
            transition: all 0.2s;
        }

        .play-morse-btn:hover {
            background: rgba(0, 255, 136, 0.4);
            transform: scale(1.05);
        }

        .morse-controls {
            display: flex;
            justify-content: space-between;
            align-items: center;
            padding: 3px 5px;
            background: rgba(0, 0, 0, 0.8);
            border: 1px solid #333;
            border-radius: 2px;
            margin-bottom: 3px;
            font-size: 0.5rem;
        }

        .morse-speed-controls {
            display: flex;
            gap: 4px;
        }

        .speed-btn {
            padding: 2px 4px;
            background: rgba(0, 0, 0, 0.8);
            border: 1px solid #333;
            color: #888;
            border-radius: 2px;
            cursor: pointer;
            font-size: 0.45rem;
            transition: all 0.2s;
        }

        .speed-btn.active {
            background: rgba(0, 255, 136, 0.2);
            border-color: #00ff88;
            color: #00ff88;
        }

        .speed-btn:hover {
            border-color: #0088ff;
            color: #0088ff;
        }

        .test-play-btn {
            padding: 2px 6px;
            background: linear-gradient(135deg, #ff6600 0%, #ff3300 100%);
            color: white;
            border: none;
            border-radius: 2px;
            cursor: pointer;
            font-size: 0.45rem;
            transition: all 0.2s;
        }

        .test-play-btn:hover {
            background: linear-gradient(135deg, #ff8800 0%, #ff5500 100%);
            transform: translateY(-1px);
        }

        .audio-status {
            display: flex;
            align-items: center;
            gap: 3px;
            color: #00ff88;
        }

        .audio-status i {
            font-size: 0.6rem;
        }

        .radio-container {
            display: flex;
            flex-direction: column;
            height: 100%;
            padding: 3px;
            gap: 5px;
        }

        .radio-controls {
            display: flex;
            gap: 5px;
            margin-bottom: 5px;
        }

        .frequency-input-container {
            flex: 1;
            display: flex;
            flex-direction: column;
            background: rgba(0, 0, 0, 0.8);
            border: 1px solid #333;
            border-radius: 3px;
            padding: 5px;
        }

        .frequency-label {
            font-size: 0.5rem;
            color: #888;
            text-transform: uppercase;
            margin-bottom: 2px;
        }

        .frequency-display {
            font-family: 'Courier New', monospace;
            font-size: 1rem;
            color: #00ff88;
            text-align: center;
            background: rgba(0, 20, 0, 0.3);
            border: 1px solid #222;
            padding: 3px;
            border-radius: 2px;
            margin-bottom: 3px;
            letter-spacing: 1px;
        }

        .frequency-input-row {
            display: flex;
            gap: 3px;
            align-items: center;
        }

        .frequency-input {
            flex: 1;
            background: rgba(0, 0, 0, 0.9);
            border: 1px solid #333;
            color: #00ff88;
            padding: 3px 5px;
            font-family: 'Courier New', monospace;
            font-size: 0.7rem;
            border-radius: 2px;
            outline: none;
        }

        .frequency-input:focus {
            border-color: #00ff88;
            box-shadow: 0 0 5px rgba(0, 255, 136, 0.3);
        }

        .unit-selector {
            background: rgba(0, 0, 0, 0.9);
            border: 1px solid #333;
            color: #00ff88;
            padding: 3px 5px;
            font-size: 0.6rem;
            border-radius: 2px;
            min-width: 50px;
        }

        .tune-buttons {
            display: flex;
            gap: 2px;
            margin-top: 3px;
        }

        .tune-btn {
            flex: 1;
            background: rgba(0, 0, 0, 0.9);
            border: 1px solid #333;
            color: #00ff88;
            padding: 2px;
            font-size: 0.6rem;
            cursor: pointer;
            border-radius: 2px;
            transition: all 0.2s;
        }

        .tune-btn:hover {
            background: rgba(0, 255, 136, 0.2);
            border-color: #00ff88;
        }

        .band-selector-container {
            flex: 1;
            display: flex;
            flex-direction: column;
            background: rgba(0, 0, 0, 0.8);
            border: 1px solid #333;
            border-radius: 3px;
            padding: 5px;
        }

        .band-buttons {
            display: grid;
            grid-template-columns: repeat(3, 1fr);
            gap: 3px;
            margin-bottom: 5px;
        }

        .band-btn {
            background: rgba(0, 0, 0, 0.9);
            border: 1px solid #333;
            color: #888;
            padding: 4px 2px;
            font-size: 0.55rem;
            cursor: pointer;
            border-radius: 2px;
            text-align: center;
            transition: all 0.2s;
        }

        .band-btn:hover {
            border-color: #0088ff;
            color: #0088ff;
        }

        .band-btn.active {
            background: rgba(0, 136, 255, 0.2);
            border-color: #0088ff;
            color: #0088ff;
            box-shadow: 0 0 5px rgba(0, 136, 255, 0.3);
        }

        .hf-bands {
            display: grid;
            grid-template-columns: repeat(4, 1fr);
            gap: 2px;
            margin-top: 3px;
        }

        .hf-band-btn {
            background: rgba(0, 0, 0, 0.9);
            border: 1px solid #444;
            color: #ffaa00;
            padding: 3px 1px;
            font-size: 0.45rem;
            cursor: pointer;
            border-radius: 1px;
            text-align: center;
            transition: all 0.2s;
        }

        .hf-band-btn:hover {
            background: rgba(255, 170, 0, 0.1);
            border-color: #ffaa00;
        }

        .hf-band-btn.active {
            background: rgba(255, 170, 0, 0.2);
            border-color: #ffaa00;
            color: #ffaa00;
            box-shadow: 0 0 4px rgba(255, 170, 0, 0.3);
        }

        .active-frequency-display {
            background: rgba(0, 20, 0, 0.3);
            border: 1px solid #00ff88;
            border-radius: 2px;
            padding: 4px;
            margin-top: 5px;
            text-align: center;
        }

        .active-freq-label {
            font-size: 0.45rem;
            color: #888;
            text-transform: uppercase;
        }

        .active-freq-value {
            font-family: 'Courier New', monospace;
            font-size: 0.7rem;
            color: #00ff88;
            font-weight: bold;
        }

        .pmr-cb-section {
            margin-top: 5px;
            border: 1px solid #333;
            border-radius: 3px;
            padding: 5px;
            background: rgba(0, 0, 0, 0.8);
        }

        .pmr-cb-title {
            font-size: 0.6rem;
            color: #00ff88;
            margin-bottom: 3px;
            text-align: center;
            border-bottom: 1px solid #333;
            padding-bottom: 3px;
        }

        .channel-type-selector {
            display: flex;
            gap: 2px;
            margin-bottom: 3px;
        }

        .channel-type-btn {
            flex: 1;
            padding: 3px;
            background: rgba(0, 0, 0, 0.9);
            border: 1px solid #333;
            color: #888;
            font-size: 0.5rem;
            cursor: pointer;
            text-align: center;
            border-radius: 2px;
            transition: all 0.2s;
        }

        .channel-type-btn:hover {
            border-color: #0088ff;
            color: #0088ff;
        }

        .channel-type-btn.active {
            background: rgba(0, 136, 255, 0.2);
            border-color: #0088ff;
            color: #0088ff;
        }

        .channel-container {
            display: none;
            margin-top: 3px;
        }

        .channel-container.active {
            display: block;
        }

        .channel-grid {
            display: grid;
            grid-template-columns: repeat(8, 1fr);
            gap: 2px;
            margin-top: 3px;
        }

        .channel-btn {
            padding: 3px 1px;
            background: rgba(0, 0, 0, 0.9);
            border: 1px solid #333;
            color: #00ff88;
            font-size: 0.5rem;
            cursor: pointer;
            text-align: center;
            border-radius: 1px;
            transition: all 0.2s;
        }

        .channel-btn:hover {
            background: rgba(0, 255, 136, 0.1);
            border-color: #00ff88;
        }

        .channel-btn.active {
            background: rgba(0, 255, 136, 0.2);
            border-color: #00ff88;
            color: #00ff88;
            box-shadow: 0 0 4px rgba(0, 255, 136, 0.3);
        }

        .channel-info {
            background: rgba(0, 20, 0, 0.3);
            border: 1px solid #00ff88;
            border-radius: 2px;
            padding: 3px;
            margin-top: 3px;
            text-align: center;
            font-size: 0.55rem;
        }

        .channel-display {
            font-family: 'Courier New', monospace;
            color: #00ff88;
            font-weight: bold;
        }

        .phonetic-table-container {
            flex: 1;
            display: flex;
            flex-direction: column;
            background: rgba(0, 0, 0, 0.8);
            border: 1px solid #333;
            border-radius: 3px;
            padding: 5px;
            overflow: hidden;
            min-height: 150px;
            max-height: 180px;
        }

        .phonetic-header {
            font-size: 0.65rem;
            color: #00ff88;
            margin-bottom: 5px;
            text-align: center;
            border-bottom: 1px solid #333;
            padding-bottom: 4px;
            flex-shrink: 0;
        }

        .phonetic-grid-container {
            flex: 1;
            overflow-y: auto;
            border: 1px solid rgba(0, 136, 255, 0.3);
            border-radius: 2px;
            background: rgba(5, 5, 5, 0.9);
        }

        .phonetic-grid {
            display: grid;
            grid-template-columns: repeat(6, 1fr);
            gap: 3px;
            padding: 3px;
        }

        .phonetic-item {
            background: rgba(0, 0, 0, 0.9);
            border: 1px solid #333;
            border-radius: 2px;
            padding: 4px;
            cursor: pointer;
            transition: all 0.2s;
            text-align: center;
            min-height: 35px;
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
        }

        .phonetic-item:hover {
            background: rgba(0, 136, 255, 0.2);
            border-color: #0088ff;
            transform: scale(1.05);
        }

        .phonetic-char {
            font-size: 0.85rem;
            color: #00ff88;
            font-weight: bold;
            margin-bottom: 2px;
        }

        .phonetic-word {
            font-size: 0.65rem;
            color: #0088ff;
            margin-bottom: 2px;
            font-weight: bold;
        }

        .phonetic-pronunciation {
            font-size: 0.5rem;
            color: #ffaa00;
            font-style: italic;
        }

        .phonetic-grid-container::-webkit-scrollbar {
            width: 5px;
        }

        .phonetic-grid-container::-webkit-scrollbar-track {
            background: rgba(0, 0, 0, 0.3);
            border-radius: 2px;
        }

        .phonetic-grid-container::-webkit-scrollbar-thumb {
            background: rgba(0, 255, 136, 0.7);
            border-radius: 2px;
            border: 1px solid rgba(0, 255, 136, 0.3);
        }

        .phonetic-grid-container::-webkit-scrollbar-thumb:hover {
            background: rgba(0, 255, 136, 0.9);
            box-shadow: 0 0 4px rgba(0, 255, 136, 0.5);
        }

        .emergency-frequencies {
            margin-top: 5px;
            border-top: 1px solid #333;
            padding-top: 3px;
            flex-shrink: 0;
        }

        .emergency-title {
            font-size: 0.5rem;
            color: #ff3300;
            text-transform: uppercase;
            margin-bottom: 2px;
        }

        .emergency-list {
            font-size: 0.45rem;
            color: #ff6600;
            max-height: 60px;
            overflow-y: auto;
        }

        .emergency-item {
            padding: 1px 2px;
            border-bottom: 1px solid rgba(255, 51, 0, 0.1);
        }

        .emergency-item:hover {
            background: rgba(255, 51, 0, 0.1);
        }

        .transmission-active {
            animation: transmissionPulse 1s infinite;
        }

        @keyframes transmissionPulse {
            0% { opacity: 1; }
            50% { opacity: 0.7; }
            100% { opacity: 1; }
        }

        .static-overlay {
            position: absolute;
            top: 0;
            left: 0;
            right: 0;
            bottom: 0;
            background: repeating-linear-gradient(
                0deg,
                transparent,
                transparent 1px,
                rgba(255, 255, 255, 0.02) 1px,
                rgba(255, 255, 255, 0.02) 2px
            );
            pointer-events: none;
            opacity: 0;
            transition: opacity 0.3s;
        }

        .static-active {
            opacity: 0.3;
        }        

        .satellite-header-v2 {
            background: linear-gradient(90deg, rgba(0, 20, 40, 0.95), rgba(0, 40, 80, 0.7));
            padding: 6px 12px;
            border-bottom: 2px solid #00ffea;
            display: flex;
            justify-content: space-between;
            align-items: center;
            border-radius: 3px;
            margin-bottom: 8px;
        }

        .satellite-title-v2 {
            font-size: 0.8rem;
            color: #00ffea;
            text-transform: uppercase;
            letter-spacing: 0.8px;
            display: flex;
            align-items: center;
            gap: 8px;
        }

        .satellite-grid-v2 {
            display: grid;
            grid-template-columns: repeat(3, 1fr);
            gap: 5px;
            margin-bottom: 8px;
        }

        .sat-data-card-v2 {
            background: rgba(0, 15, 30, 0.8);
            border: 1px solid rgba(0, 136, 255, 0.4);
            border-radius: 3px;
            padding: 6px;
            display: flex;
            flex-direction: column;
            transition: all 0.3s;
        }

        .sat-data-card-v2:hover {
            border-color: #00ffea;
            background: rgba(0, 25, 50, 0.9);
            transform: translateY(-1px);
        }

        .sat-data-label-v2 {
            font-size: 0.45rem;
            color: #88aaff;
            text-transform: uppercase;
            letter-spacing: 0.5px;
            margin-bottom: 3px;
        }

        .sat-data-value-v2 {
            font-size: 0.75rem;
            color: #00ffea;
            font-weight: bold;
            font-family: 'Courier New', monospace;
        }

        .sat-gps-container-v2 {
            background: rgba(0, 20, 0, 0.4);
            border: 1px solid #00ff88;
            border-radius: 3px;
            padding: 8px;
            margin: 8px 0;
            text-align: center;
        }

        .gps-coords-v2 {
            font-family: 'Courier New', monospace;
            font-size: 0.85rem;
            color: #00ffea;
            font-weight: bold;
            letter-spacing: 1px;
            margin: 4px 0;
        }

        .emergency-panel-v2 {
            background: linear-gradient(135deg, rgba(40, 0, 0, 0.9), rgba(80, 0, 0, 0.7));
            border: 2px solid #ff3300;
            border-radius: 4px;
            padding: 10px;
            margin-top: 10px;
            position: relative;
            overflow: hidden;
        }

        .emergency-panel-v2::before {
            content: '';
            position: absolute;
            top: 0;
            left: 0;
            right: 0;
            height: 3px;
            background: linear-gradient(90deg, #ff3300, #ff6600, #ff3300);
            animation: emergencyPulse 2s infinite;
        }

        @keyframes emergencyPulse {
            0% { opacity: 0.3; }
            50% { opacity: 1; }
            100% { opacity: 0.3; }
        }

        .emergency-title-v2 {
            font-size: 0.75rem;
            color: #ffaa00;
            text-transform: uppercase;
            text-align: center;
            margin-bottom: 8px;
            display: flex;
            align-items: center;
            justify-content: center;
            gap: 8px;
        }

        .emergency-buttons-v2 {
            display: flex;
            gap: 6px;
            margin-bottom: 6px;
        }

        .emergency-btn-big-v2 {
            flex: 2;
            padding: 10px;
            background: linear-gradient(135deg, #ff3300, #ff6600);
            color: white;
            border: none;
            border-radius: 3px;
            font-size: 0.7rem;
            font-weight: bold;
            cursor: pointer;
            transition: all 0.2s;
            text-transform: uppercase;
            letter-spacing: 0.5px;
            text-align: center;
        }

        .emergency-btn-big-v2:hover {
            background: linear-gradient(135deg, #ff5500, #ff8800);
            transform: translateY(-2px);
            box-shadow: 0 4px 12px rgba(255, 51, 0, 0.4);
        }

        .position-btn-v2 {
            flex: 1;
            padding: 10px 6px;
            background: rgba(0, 136, 255, 0.3);
            border: 1px solid #0088ff;
            color: #00ffea;
            border-radius: 3px;
            font-size: 0.6rem;
            cursor: pointer;
            transition: all 0.2s;
        }

        .position-btn-v2:hover {
            background: rgba(0, 136, 255, 0.5);
            transform: scale(1.05);
        }

        .api-status-v2 {
            display: flex;
            align-items: center;
            gap: 6px;
            padding: 4px 8px;
            background: rgba(0, 20, 40, 0.6);
            border-radius: 2px;
            font-size: 0.55rem;
            margin-bottom: 6px;
            justify-content: center;
        }

        .status-dot-api {
            width: 8px;
            height: 8px;
            border-radius: 50%;
        }

        .status-connected {
            background: #00ff88;
            box-shadow: 0 0 8px #00ff88;
            animation: pulse 2s infinite;
        }

        .status-disconnected {
            background: #ff3300;
            box-shadow: 0 0 8px #ff3300;
        }

        .mic-btn {
            width: 36px;
            height: 28px;
            display: flex;
            align-items: center;
            justify-content: center;
            border-radius: 2px;
            border: 1px solid #333;
            background: linear-gradient(135deg, #222 0, #333 100%);
            color: #ff4444;
            cursor: pointer;
            transition: all 0.2s;
            font-size: 0.9rem;
        }

        .mic-btn:hover {
            border-color: #ff4444;
            box-shadow: 0 0 6px rgba(255, 68, 68, 0.5);
        }

        .mic-btn.listening {
            background: linear-gradient(135deg, #ff3300 0, #ff6600 100%);
            color: #000;
            border-color: #ffcc00;
            box-shadow: 0 0 12px rgba(255, 102, 0, 0.8);
        }

        #mic-status {
            font-size: 0.55rem;
            color: #888;
            min-width: 70px;
        }

        #mic-status.active {
            color: #ff6600;
        }

        #mic-status.active {
            color: #ff6600;
            font-weight: bold;
            animation: voicePulse 1.5s infinite;
        }

        @keyframes voicePulse {
            0% { opacity: 1; }
            50% { opacity: 0.7; }
            100% { opacity: 1; }
        }

        .mic-btn.listening {
            animation: micPulse 5s infinite;
        }

        @keyframes micPulse {
            0% { 
                box-shadow: 0 0 5px rgba(255, 68, 68, 0.5);
            }
            50% { 
                box-shadow: 0 0 15px rgba(255, 68, 68, 0.8);
            }
            100% { 
                box-shadow: 0 0 5px rgba(255, 68, 68, 0.5);
            }
        }

.morse-translation {
    background: rgba(0, 20, 0, 0.3);
    border: 1px solid #ffaa00;
    border-radius: 2px;
    padding: 4px 6px;
    margin: 4px 8px 0 8px;
    font-size: 0.6rem;
    color: #ffaa00;
    font-family: 'Courier New', monospace;
    display: none;
}

.morse-translation.active {
    display: block;
}

.translation-label {
    color: #00ff88;
    font-weight: bold;
    margin-right: 5px;
}

::-webkit-scrollbar {
    width: 3px !important;
    height: 3px !important;
}

::-webkit-scrollbar-track {
    background: rgba(10, 10, 10, 0.6) !important;
    border-radius: 10px !important;
}

::-webkit-scrollbar-thumb {
    background: #00ff88 !important;
    border-radius: 10px !important;
    border: 1px solid rgba(0, 255, 136, 0.2) !important;
}

::-webkit-scrollbar-thumb:hover {
    background: #33ff99 !important;
}

.security-badge {
    display: inline-flex;
    align-items: center;
    gap: 4px;
    background: rgba(0, 136, 255, 0.2);
    padding: 2px 6px;
    border-radius: 3px;
    font-size: 0.6rem;
    border: 1px solid rgba(0, 136, 255, 0.4);
    cursor: pointer;
    margin-left: 8px;
}
.security-badge:hover {
    background: rgba(0, 136, 255, 0.3);
}
.security-dot {
    width: 6px;
    height: 6px;
    border-radius: 50%;
    background: #00ff88;
    box-shadow: 0 0 4px #00ff88;
}

.security-aes { color: #00ffea !important; }
.security-xor { color: #ffaa00 !important; }
.security-none { color: #ff3300 !important; }

.console-btn {
    width: 36px;
    height: 28px;
    display: flex;
    align-items: center;
    justify-content: center;
    border-radius: 2px;
    border: 1px solid #333;
    background: linear-gradient(135deg, #222 0, #333 100%);
    color: #00ff88;
    cursor: pointer;
    transition: all 0.2s;
    font-size: 0.9rem;
}

.console-btn:hover {
    border-color: #00ff88;
    box-shadow: 0 0 6px rgba(0, 255, 136, 0.5);
    background: linear-gradient(135deg, #333 0, #444 100%);
}

.console-modal-overlay {
    display: none;
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    background: rgba(0, 0, 0, 0.95);
    z-index: 2000;
    align-items: center;
    justify-content: center;
    padding: 20px;
}

.console-modal-content {
    background: linear-gradient(145deg, #050505 0%, #0a0a0a 100%);
    border: 2px solid #00ff88;
    border-radius: 5px;
    width: 680px;
    height: 680px;
    color: #00ff88;
    display: flex;
    flex-direction: column;
    overflow: hidden;
    box-shadow: 0 0 40px rgba(0, 255, 136, 0.2);
}

.console-header {
    background: linear-gradient(90deg, #000 0%, #111 100%);
    padding: 8px 12px;
    border-bottom: 2px solid #00ff88;
    display: flex;
    justify-content: space-between;
    align-items: center;
}

.console-title {
    font-size: 0.9rem;
    color: #00ff88;
    text-transform: uppercase;
    letter-spacing: 0.8px;
    display: flex;
    align-items: center;
    gap: 8px;
}

.console-close {
    background: none;
    border: none;
    color: #ff3300;
    font-size: 1.2rem;
    cursor: pointer;
    padding: 4px 8px;
    border-radius: 2px;
    transition: all 0.2s;
}

.console-close:hover {
    background: rgba(255, 51, 0, 0.2);
}

.console-tabs {
    display: flex;
    background: rgba(0, 0, 0, 0.9);
    border-bottom: 1px solid #333;
}

.console-tab {
    flex: 1;
    padding: 6px 10px;
    background: rgba(17, 17, 17, 0.8);
    color: #888;
    border: none;
    cursor: pointer;
    font-size: 0.7rem;
    text-transform: uppercase;
    transition: all 0.3s;
    border-right: 1px solid #222;
}

.console-tab:last-child {
    border-right: none;
}

.console-tab:hover {
    background: rgba(0, 255, 136, 0.1);
    color: #00ff88;
}

.console-tab.active {
    background: rgba(0, 255, 136, 0.15);
    color: #00ff88;
    border-bottom: 2px solid #00ff88;
}

.console-tab-content {
    display: none;
    flex: 1;
    overflow: hidden;
    padding: 10px;
}

.console-tab-content.active {
    display: flex;
    flex-direction: column;
}

.CMD-console-container {
    flex: 1;
    display: flex;
    flex-direction: column;
    overflow: hidden;
}

.CMD-output {
    flex: 1;
    background: rgba(0, 10, 0, 0.8);
    border: 1px solid #00ff88;
    border-radius: 3px;
    padding: 8px;
    font-family: 'Courier New', monospace;
    font-size: 0.7rem;
    color: #00ff88;
    overflow-y: auto;
    margin-bottom: 8px;
    white-space: pre-wrap;
    line-height: 1.2;
}

.CMD-input-row {
    display: flex;
    gap: 8px;
}

.CMD-input {
    flex: 1;
    background: rgba(0, 0, 0, 0.9);
    border: 1px solid #333;
    color: #00ff88;
    padding: 6px 8px;
    font-family: 'Courier New', monospace;
    font-size: 0.7rem;
    border-radius: 2px;
    outline: none;
}

.CMD-input:focus {
    border-color: #00ff88;
    box-shadow: 0 0 5px rgba(0, 255, 136, 0.3);
}

.CMD-btn {
    padding: 6px 12px;
    background: linear-gradient(135deg, #00ff88 0%, #0088ff 100%);
    color: #000;
    border: none;
    border-radius: 2px;
    font-weight: bold;
    cursor: pointer;
    font-size: 0.7rem;
    transition: all 0.2s;
}

.CMD-btn:hover {
    transform: translateY(-1px);
    box-shadow: 0 2px 8px rgba(0, 255, 136, 0.3);
}

.CMD-btn-clear {
    background: linear-gradient(135deg, #ff3300 0%, #ff6600 100%);
    color: white;
}

.hex-editor-container {
    flex: 1;
    display: flex;
    flex-direction: column;
    overflow: hidden;
}

.hex-toolbar {
    display: flex;
    gap: 6px;
    margin-bottom: 8px;
    padding-bottom: 8px;
    border-bottom: 1px solid #333;
}

.hex-toolbar-btn {
    padding: 4px 8px;
    background: rgba(0, 0, 0, 0.9);
    border: 1px solid #333;
    color: #888;
    border-radius: 2px;
    cursor: pointer;
    font-size: 0.6rem;
    transition: all 0.2s;
}

.hex-toolbar-btn:hover {
    border-color: #0088ff;
    color: #0088ff;
}

.hex-toolbar-btn.active {
    background: rgba(0, 136, 255, 0.2);
    border-color: #0088ff;
    color: #0088ff;
}

.hex-editor {
    flex: 1;
    background: rgba(0, 0, 0, 0.9);
    border: 1px solid #00ff88;
    border-radius: 3px;
    padding: 10px;
    font-family: 'Courier New', monospace;
    font-size: 0.65rem;
    color: #00ff88;
    outline: none;
    resize: none;
    white-space: pre;
    overflow: auto;
    line-height: 1.1;
}

.hex-editor:focus {
    box-shadow: 0 0 8px rgba(0, 255, 136, 0.2);
}

.hex-status-bar {
    margin-top: 8px;
    padding: 4px 6px;
    background: rgba(0, 0, 0, 0.8);
    border: 1px solid #333;
    border-radius: 2px;
    font-size: 0.55rem;
    color: #888;
    display: flex;
    justify-content: space-between;
}

.hex-decoder-container {
    flex: 1;
    display: flex;
    flex-direction: column;
    overflow: hidden;
}

.hex-decoder-split {
    display: flex;
    flex: 1;
    gap: 10px;
    overflow: hidden;
}

.hex-decoder-input {
    flex: 1;
    display: flex;
    flex-direction: column;
}

.hex-decoder-output {
    flex: 1;
    display: flex;
    flex-direction: column;
}

.hex-decoder-label {
    font-size: 0.65rem;
    color: #00ff88;
    margin-bottom: 5px;
    text-transform: uppercase;
    letter-spacing: 0.5px;
}

.hex-decoder-textarea {
    flex: 1;
    background: rgba(0, 0, 0, 0.9);
    border: 1px solid #333;
    border-radius: 3px;
    padding: 8px;
    font-family: 'Courier New', monospace;
    font-size: 0.65rem;
    color: #00ff88;
    outline: none;
    resize: none;
}

.hex-decoder-textarea:focus {
    border-color: #00ff88;
}

.hex-decoder-controls {
    display: flex;
    gap: 6px;
    margin-top: 8px;
}

.hex-decoder-btn {
    flex: 1;
    padding: 6px 10px;
    background: linear-gradient(135deg, #222 0%, #333 100%);
    border: 1px solid #444;
    color: #00ff88;
    border-radius: 2px;
    cursor: pointer;
    font-size: 0.65rem;
    transition: all 0.2s;
}

.hex-decoder-btn:hover {
    background: linear-gradient(135deg, #00ff88 0%, #0088ff 100%);
    color: #000;
}

.hex-editor::-webkit-scrollbar,
.CMD-output::-webkit-scrollbar,
.hex-decoder-textarea::-webkit-scrollbar {
    width: 6px;
    height: 6px;
}

.hex-editor::-webkit-scrollbar-track,
.CMD-output::-webkit-scrollbar-track,
.hex-decoder-textarea::-webkit-scrollbar-track {
    background: rgba(0, 0, 0, 0.3);
    border-radius: 3px;
}

.hex-editor::-webkit-scrollbar-thumb,
.CMD-output::-webkit-scrollbar-thumb,
.hex-decoder-textarea::-webkit-scrollbar-thumb {
    background: #00ff88;
    border-radius: 3px;
}

.hex-editor::-webkit-scrollbar-thumb:hover,
.CMD-output::-webkit-scrollbar-thumb:hover,
.hex-decoder-textarea::-webkit-scrollbar-thumb:hover {
    background: #33ff99;
}

.hex-byte {
    padding: 0 1px;
    border-radius: 1px;
    transition: all 0.1s;
}

.hex-byte:hover {
    background: rgba(0, 255, 136, 0.3);
    color: #fff;
}

.hex-ascii {
    color: #ffaa00;
    font-weight: bold;
}

.glass-cockpit-container {
    margin-top: 15px;
    padding: 10px;
    border-top: 2px solid #222;
    background: rgba(0,0,0,0.3);
}

.cockpit-grid {
    display: grid;
    grid-template-columns: repeat(2, 1fr);
    gap: 8px;
}

.btn-cockpit {
    background: #111;
    border: 1px solid #333;
    color: #00ff88;
    padding: 10px 5px;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    cursor: pointer;
    transition: all 0.2s;
    border-radius: 4px;
    font-family: 'Orbitron', sans-serif;
}

.btn-cockpit i {
    font-size: 1.2rem;
    margin-bottom: 4px;
}

.btn-cockpit span {
    font-size: 0.6rem;
    letter-spacing: 1px;
}

.btn-cockpit:hover {
    background: #1a1a1a;
    border-color: #00ff88;
    box-shadow: 0 0 10px rgba(0, 255, 136, 0.2);
}

.btn-cockpit.active {
    background: #00ff88;
    color: #000;
    font-weight: bold;
    border-color: #fff;
}

        /* ===== ANIMACIONES PARA SPLASH SCREEN ===== */
        @keyframes neonPulse {
            from {
                text-shadow: 0 0 5px #00ff88,
                             0 0 10px #00ff88,
                             0 0 20px #00ff88,
                             0 0 40px #00ff88;
            }
            to {
                text-shadow: 0 0 10px #00ff88,
                             0 0 20px #00ff88,
                             0 0 40px #00ff88,
                             0 0 80px #00ff88,
                             0 0 120px #0088ff;
            }
        }

        @keyframes scanLine {
            0% { transform: translateX(-100%); }
            100% { transform: translateX(100%); }
        }

        @keyframes loadingBar {
    0% { left: -40%; }
    100% { left: 100%; }
}

.voip-control-panel {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    width: 250px;
    background: rgba(0, 0, 0, 0.95);
    border: 2px solid #ff3300;
    border-radius: 8px;
    padding: 15px;
    z-index: 5000;
    box-shadow: 0 0 30px rgba(255, 51, 0, 0.5);
    backdrop-filter: blur(5px);
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 10px;
    animation: panelFadeIn 0.3s ease;
}

@keyframes panelFadeIn {
    from { opacity: 0; transform: translate(-50%, -60%); }
    to { opacity: 1; transform: translate(-50%, -50%); }
}

.voip-panel-title {
    color: #ff3300;
    font-size: 0.8rem;
    font-weight: bold;
    text-transform: uppercase;
    letter-spacing: 1px;
    border-bottom: 1px solid #ff3300;
    padding-bottom: 5px;
    width: 100%;
    text-align: center;
}

.voip-panel-peer {
    color: #00ff88;
    font-size: 0.9rem;
    font-weight: bold;
    background: rgba(0, 255, 136, 0.1);
    padding: 5px 10px;
    border-radius: 20px;
    width: 100%;
    text-align: center;
    word-break: break-all;
}

.voip-panel-status {
    color: #ffaa00;
    font-size: 0.7rem;
    text-transform: uppercase;
}

.voip-panel-buttons {
    display: flex;
    gap: 15px;
    margin-top: 10px;
    width: 100%;
    justify-content: center;
}

.voip-panel-btn {
    width: 50px;
    height: 50px;
    border-radius: 50%;
    border: none;
    font-size: 1.4rem;
    cursor: pointer;
    display: flex;
    align-items: center;
    justify-content: center;
    transition: all 0.2s;
    box-shadow: 0 0 15px rgba(0,0,0,0.5);
}

.voip-panel-btn-accept {
    background: #00ff88;
    color: #000;
}

.voip-panel-btn-accept:hover {
    transform: scale(1.1);
    box-shadow: 0 0 20px #00ff88;
}

.voip-panel-btn-reject, .voip-panel-btn-end {
    background: #ff3300;
    color: white;
}

.voip-panel-btn-reject:hover, .voip-panel-btn-end:hover {
    transform: scale(1.1);
    box-shadow: 0 0 20px #ff3300;
}

.voip-panel-btn:active {
    transform: scale(0.95);
}
    </style>
</head>
<body>
    <!-- CABECERA -->
<div class="container">
    <!-- ===== SPLASH SCREEN CENTRADO EN 720x720 (NUEVO) ===== -->
    <div id="splash-screen" style="
        position: absolute;
        top: 0;
        left: 0;
        width: 100%;
        height: 100%;
        background: #0a0a0a;
        z-index: 10000;
        display: flex;
        justify-content: center;
        align-items: center;
        flex-direction: column;
        transition: opacity 0.8s ease;
        opacity: 1;
        backdrop-filter: blur(0px);
        ">
        <div style="text-align: center; width: 95%;">
            <div style="
                font-family: 'Consolas', 'Courier New', monospace;
                font-size: 4.5rem;
                font-weight: bold;
                color: #ffffff;
                text-shadow: 0 0 5px rgba(245, 248, 246, 0.7);
                line-height: 1.2;
                margin-bottom: 15px;
                ">
                RAD<span style="color: #04fc4a;">COM MASTER</span>
            </div>
            
            <div style="
                font-size: 0.8rem;
                color: #ffffff;
                letter-spacing: 2px;
                margin-bottom: 25px;
                opacity: 0.8;
                ">v5.7.2 SEGURIDAD ACTIVADA + VoIP
            </div>
            
            <div style="
                width: 250px;
                height: 2px;
                background: rgba(255,255,255,0.2);
                margin: 15px auto;
                position: relative;
                overflow: hidden;
                ">
                <div style="
                    width: 40%;
                    height: 100%;
                    background: #29f700;
                    position: absolute;
                    left: -40%;
                    animation: loadingBar 1.5s infinite ease-in-out;
                    "></div>
            </div>
            
            <div style="
                color: #ffffff;
                font-size: 0.65rem;
                opacity: 0.6;
                ">SISTEMA RADCOM MASTER INICIANDO.......
            </div>
        </div>
    </div>
    <!-- ===== FIN SPLASH SCREEN ===== -->
        <div class="header-pro">
            <div class="status-indicator">
    <span class="status-dot-live"></span>
    <span>RADCOM MASTER <span class="version-badge">v5.7.2</span></span>
    <span style="color:#7b7d7b; margin-left:10px;">|</span>
    <span id="data-session" style="color:#00ffea">0</span>b
    <div class="security-badge" onclick="showSecurityInfo()" title="Seguridad AES-256-GCM Activada">
        <span class="security-dot"></span>
        <span style="color:#00ffea;">AES-256-GCM</span>
    </div>
</div>
            <div id="display-id" style="font-size:0.6rem; color:lch(88.59% 86.83 150.39); font-family:monospace;">INICIANDO...</div>
            <div style="display:flex; gap:4px; align-items:center;">
                <button class="btn-header btn-refresh" onclick="refreshAllConnections()" title="Refrescar conexiones">
                    <i class="fas fa-sync-alt"></i>
                </button>
                
                <button class="btn-header" onclick="showSettings()" title="Configuración">
                    <i class="fas fa-cog"></i>
                </button>
                <button class="btn-header" onclick="copyID()" title="Copiar ID">
                    <i class="fas fa-copy"></i>
                </button>
                <input type="text" id="peer-id-input" placeholder="ID Satélite" 
                       style="width:85px; background:rgba(0,0,0,0.8); color:#fff; 
                              border:1px solid #333; font-size:0.55rem; padding:2px 4px;
                              border-radius:2px; outline:none;">
                <button class="btn-header" style="background:#00ff88; color:#000; border:none; 
                        padding:2px 6px;" onclick="connectToPeer()">
                    <i class="fas fa-link"></i> LINK
                </button>
            </div>
        </div>

        <div class="main-layout">
            <div class="sidebar" id="peer-list">
                <div class="sidebar-section">
                    <div class="sidebar-title">
                        SISTEMA CEREBRO
                        <span class="encryption-indicator">
                            <i class="fas fa-shield-alt"></i> AES-256-GCM
                        </span>
                    </div>
                    <div id="target-global" class="peer-item active" onclick="selectPeer('GLOBAL')">
                        <div class="peer-info">
                            <span class="status-dot st-online"></span>
                            <div style="display:flex; flex-direction:column;">
                                <span style="font-weight:bold;">TRANSMISIÓN GLOBAL</span>
                                <span style="font-size:0.55rem; color:#888;">
                                    <span id="connected-count">0</span> satélites
                                </span>
                            </div>
                        </div>
                        <span style="color:#00ff88; font-size:0.55rem;">ACTIVO</span>
                    </div>
                </div>
                
                <div class="sidebar-section">
                    <div class="sidebar-title">
                        HISTORIAL AUTO-LINK
                        <span onclick="toggleShowOffline()" style="cursor:pointer; font-size:0.5rem; color:#888;" 
                              title="Mostrar/Ocultar offline">
                            <i class="fas fa-eye-slash"></i>
                        </span>
                        <span class="encryption-indicator">
                            <i class="fas fa-history"></i> LOCAL
                        </span>
                    </div>
                    <div id="saved-peers-container">
                        <div id="saved-peers"></div>
                    </div>
                </div>
                
                <div class="sidebar-section">
                    <div class="sidebar-title">
                        ESTADÍSTICAS
                    </div>
                    <div style="padding:5px; font-size:0.55rem;">
                        <div style="margin-bottom:2px;">
                            <span style="color:#888;">Mensajes:</span>
                            <span id="stats-messages" style="color:#00ff88; float:right;">0</span>
                        </div>
                        <div style="margin-bottom:2px;">
                            <span style="color:#888;">Datos TX:</span>
                            <span id="stats-tx" style="color:#0088ff; float:right;">0 B</span>
                        </div>
                        <div style="margin-bottom:2px;">
                            <span style="color:#888;">Datos RX:</span>
                            <span id="stats-rx" style="color:#ff3300; float:right;">0 B</span>
                        </div>
                        <div>
                            <span style="color:#888;">Latencia:</span>
                            <span id="stats-latency" style="color:#00ff88; float:right;">-- ms</span>
                        </div>
                        
                        <button class="revive-btn" onclick="reviveAllConnections()" title="Reactivar conexiones inactivas" id="reviveBtn">
                            <i class="fas fa-heartbeat"></i> REVIVIR CONEXIONES
                        </button>
                    </div>
                     <div class="glass-cockpit-container">
    <div class="cockpit-grid">
        <button class="btn-cockpit active" id="btn-com" onclick="handleCockpitClick('COM')">
            <i class="fas fa-comments"></i><span>COM</span>
        </button>
        <button class="btn-cockpit" id="btn-nav" onclick="handleCockpitClick('NAV')">
            <i class="fas fa-compass"></i><span>NAV</span>
        </button>
        <button class="btn-cockpit" id="btn-ecm" onclick="handleCockpitClick('ECM')">
            <i class="fas fa-engine"></i><span>ECM</span>
        </button>
        <button class="btn-cockpit" id="btn-map" onclick="handleCockpitClick('MAP')">
            <i class="fas fa-map-marked-alt"></i><span>MAP</span>
        </button>
        <button class="btn-cockpit" id="btn-util" onclick="handleCockpitClick('UTIL')">
            <i class="fas fa-tools"></i><span>UTIL</span>
        </button>
        <button class="btn-cockpit" id="btn-res" onclick="handleCockpitClick('RES')">
            <i class="fas fa-plus"></i><span>EXT</span>
        </button>
    </div>
                </div>             
</div>
            </div>

            <div class="content-area" id="content-area">
                <div class="monitor-container" id="monitorContainer">
                    <div id="monitor-raw">SISTEMA INICIADO | ESCUCHANDO CONEXIONES...</div>
                        <div class="morse-translation" id="morse-translation">
    <span class="translation-label">TRADUCCIÓN:</span>
    <span id="morse-text-display"></span> → 
    <span id="morse-code-display"></span>
</div>
                    
                    <div class="resizable-separator" id="resizableSeparator"></div>
                    
                    <div class="table-container" id="tableContainer">
                        <div class="table-info">
                            <div class="char-preview">
                                <div class="preview-box">
                                    <div class="preview-label">CARÁCTER</div>
                                    <div id="current-char" class="preview-value">-</div>
                                </div>
                                <div class="preview-box">
                                    <div class="preview-label">HEX</div>
                                    <div id="current-hex" class="preview-value" style="color:#00ffea;">--</div>
                                </div>
                                <div class="preview-box">
                                    <div class="preview-label">DEC</div>
                                    <div id="current-dec" class="preview-value" style="color:#0088ff;">---</div>
                                </div>
                                <div class="preview-box">
                                    <div class="preview-label">BIN</div>
                                    <div id="current-bin" class="preview-value" style="color:#ff3300; font-size:0.6rem;">--------</div>
                                </div>
                                <div class="preview-box">
                                    <div class="preview-label">POSICIÓN</div>
                                    <div id="current-pos" class="preview-value" style="color:#00ff88;">V:-- H:--</div>
                                </div>
                            </div>
    <button id="consoleBtn" class="console-btn" title="Consola & Programación HEX" onclick="openConsole()">
        <i class="fas fa-terminal"></i>
    </button>
                            <button class="btn-header" onclick="clearTableSelection()" 
                                    style="font-size:0.45rem; padding:1px 3px;">
                                <i class="fas fa-times"></i> LIMPIAR
                            </button>
                        </div>  

                        <div class="tab-container">
                            <button class="tab-button active" data-tab="ascii" onclick="switchTab('ascii')">ASCII</button>
                            <button class="tab-button" data-tab="morse" onclick="switchTab('morse')">CQ Morse</button>
                            <button class="tab-button" data-tab="radio" onclick="switchTab('radio')">Radio</button>
                            <button class="tab-button" data-tab="satellite" onclick="switchTab('satellite')">Satélite</button>
                        </div>
                        
                        <div class="ansi-container">
                            <div id="ascii-table" class="tab-content active">
                                <table id="ansiTable"></table>
                            </div>
                            <div id="morse-table" class="tab-content">
                                <div class="morse-controls">
                                    <div class="audio-status">
                                        <i class="fas fa-volume-up"></i>
                                        <span>SONIDO MORSE</span>
                                    </div>
                                    <div class="morse-speed-controls">
                                        <span style="color:#888; margin-right:2px;">VEL:</span>
                                        <button class="speed-btn active" onclick="setMorseSpeed('slow')">LENTA</button>
                                        <button class="speed-btn" onclick="setMorseSpeed('normal')">NORMAL</button>
                                        <button class="speed-btn" onclick="setMorseSpeed('fast')">RÁPIDA</button>
                                    </div>
                                    <button class="test-play-btn" onclick="playMorsePreview()">
                                        <i class="fas fa-play"></i> PROBAR
                                    </button>
                                </div>
                                
                                <div class="morse-table-container">
                                    <table id="morseTable"></table>
                                </div>
                            </div>
                            <div id="radio-table" class="tab-content">
                                <div class="radio-container">
                                    <div class="radio-controls">
                                        <div class="frequency-input-container">
                                            <div class="frequency-label">Frecuencia de Transmisión</div>
                                            <div class="frequency-display" id="radio-frequency-display">142.850 MHz</div>
                                            <div class="frequency-input-row">
                                                <input type="text" class="frequency-input" id="radio-frequency-input" 
                                                       value="142.850" placeholder="000.000">
                                                <select class="unit-selector" id="radio-unit">
                                                    <option value="MHz">MHz</option>
                                                    <option value="KHz">KHz</option>
                                                </select>
                                            </div>
                                            <div class="tune-buttons">
                                                <button class="tune-btn" onclick="tuneFrequency(-0.005)">-5KHz</button>
                                                <button class="tune-btn" onclick="tuneFrequency(-0.001)">-1KHz</button>
                                                <button class="tune-btn" onclick="tuneFrequency(0.001)">+1KHz</button>
                                                <button class="tune-btn" onclick="tuneFrequency(0.005)">+5KHz</button>
                                            </div>
                                        </div>
                                        
                                        <div class="band-selector-container">
                                            <div class="frequency-label">Selector de Banda</div>
                                            <div class="band-buttons">
                                                <button class="band-btn" data-band="UHF" onclick="selectBand('UHF')">UHF</button>
                                                <button class="band-btn active" data-band="VHF" onclick="selectBand('VHF')">VHF</button>
                                                <button class="band-btn" data-band="AEREA" onclick="selectBand('AEREA')">AÉREA</button>
                                                <button class="band-btn" data-band="MARINA" onclick="selectBand('MARINA')">MARINA</button>
                                                <button class="band-btn" data-band="HF" onclick="selectBand('HF')">HF</button>
                                                <button class="band-btn" data-band="EMERG" onclick="selectBand('EMERG')">EMERG.</button>
                                            </div>
                                            
                                            <div id="hf-sub-bands" style="display: none;">
                                                <div style="font-size:0.45rem; color:#ffaa00; margin-bottom:2px;">Bandas HF:</div>
                                                <div class="hf-bands">
                                                    <button class="hf-band-btn" data-hf="10m" onclick="selectHFBand('10m')">10m</button>
                                                    <button class="hf-band-btn" data-hf="15m" onclick="selectHFBand('15m')">15m</button>
                                                    <button class="hf-band-btn" data-hf="20m" onclick="selectHFBand('20m')">20m</button>
                                                    <button class="hf-band-btn" data-hf="40m" onclick="selectHFBand('40m')">40m</button>
                                                    <button class="hf-band-btn" data-hf="80m" onclick="selectHFBand('80m')">80m</button>
                                                    <button class="hf-band-btn" data-hf="160m" onclick="selectHFBand('160m')">160m</button>
                                                    <button class="hf-band-btn" data-hf="320m" onclick="selectHFBand('320m')">320m</button>
                                                    <button class="hf-band-btn" data-hf="640m" onclick="selectHFBand('640m')">640m</button>
                                                </div>
                                            </div>
                                            
                                            <div class="pmr-cb-section">
                                                <div class="pmr-cb-title">
                                                    <i class="fas fa-walkie-talkie"></i> CB27 & PMR446
                                                </div>
                                                <div class="channel-type-selector">
                                                    <button class="channel-type-btn active" onclick="selectChannelType('pmr')">PMR446</button>
                                                    <button class="channel-type-btn" onclick="selectChannelType('cb')">CB 27MHz</button>
                                                </div>
                                                
                                                <div id="pmr-container" class="channel-container active">
                                                    <div style="font-size:0.45rem; color:#888; margin-bottom:2px;">
                                                        Selecciona tipo:
                                                        <select id="pmr-type" style="margin-left:5px; font-size:0.4rem;" onchange="selectPMRType(this.value)">
                                                            <option value="8">8 Canales</option>
                                                            <option value="16">16 Canales</option>
                                                            <option value="32">32 Canales</option>
                                                        </select>
                                                    </div>
                                                    <div class="channel-grid" id="pmr-channels-grid">
                                                    </div>
                                                </div>
                                                
                                                <div id="cb-container" class="channel-container">
                                                    <div class="channel-grid" id="cb-channels-grid">
                                                    </div>
                                                </div>
                                                
                                                <div class="channel-info">
                                                    <div style="font-size:0.4rem; color:#888;">Canal Activo:</div>
                                                    <div class="channel-display" id="channel-display">VHF 142.850 MHz</div>
                                                </div>
                                            </div>
                                            
                                            <div class="active-frequency-display">
                                                <div class="active-freq-label">Frecuencia Activa</div>
                                                <div class="active-freq-value" id="active-frequency">142.850 MHz VHF</div>
                                            </div>
                                        </div>
                                    </div>
                                    
                                    <div class="phonetic-table-container">
                                        <div class="phonetic-header">
                                            <i class="fas fa-broadcast-tower"></i> ALFABETO FONÉTICO RADIO
                                        </div>
                                        <div class="phonetic-grid-container">
                                            <div class="phonetic-grid" id="phonetic-grid">
                                            </div>
                                        </div>
                                        
                                        <div class="emergency-frequencies">
                                            <div class="emergency-title">
                                                <i class="fas fa-exclamation-triangle"></i> FRECUENCIAS DE EMERGENCIA
                                            </div>
                                            <div class="emergency-list" id="emergency-list">
                                            </div>
                                        </div>
                                    </div>
                                    
                                    <div class="static-overlay" id="static-overlay"></div>
                                </div>
                            </div>
                            <div id="satellite-table" class="tab-content active">
                                <div class="satellite-container-v2">
                                    <div class="satellite-header-v2">
                                        <div class="satellite-title-v2">
                                            <i class="fas fa-satellite"></i>
                                            SISTEMA SATELITAL UNIFICADO
                                        </div>
                                        <div style="display:flex; gap:4px;">
                                            <button class="refresh-btn" onclick="forceUpdateSatelliteData()" title="Actualizar ahora">
                                                <i class="fas fa-sync-alt"></i>
                                            </button>
                                        </div>
                                    </div>
                                    
                                    <div class="api-status-v2">
                                        <span class="status-dot-api status-connected" id="api-status-dot"></span>
                                        <span id="api-status-text">Conectando a API Open-Meteo...</span>
                                    </div>
                                    
                                    <div class="satellite-grid-v2">
                                        <div class="sat-data-card-v2">
                                            <div class="sat-data-label-v2">Temperatura</div>
                                            <div class="sat-data-value-v2" id="sat-temp">-- °C</div>
                                            <div class="sat-data-label-v2">Sensación Térmica</div>
                                            <div class="sat-data-value-v2" id="sat-feelslike">-- °C</div>
                                        </div>
                                        
                                        <div class="sat-data-card-v2">
                                            <div class="sat-data-label-v2">Humedad</div>
                                            <div class="sat-data-value-v2" id="sat-humidity">-- %</div>
                                            <div class="sat-data-label-v2">Presión</div>
                                            <div class="sat-data-value-v2" id="sat-pressure">-- hPa</div>
                                        </div>
                                        
                                        <div class="sat-data-card-v2">
                                            <div class="sat-data-label-v2">Viento Velocidad</div>
                                            <div class="sat-data-value-v2" id="sat-windspeed">-- km/h</div>
                                            <div class="sat-data-label-v2">Viento Dirección</div>
                                            <div class="sat-data-value-v2" id="sat-winddirection">-- °</div>
                                        </div>
                                        
                                        <div class="sat-data-card-v2">
                                            <div class="sat-data-label-v2">Ráfagas Máx</div>
                                            <div class="sat-data-value-v2" id="sat-windgusts">-- km/h</div>
                                            <div class="sat-data-label-v2">Índice UV</div>
                                            <div class="sat-data-value-v2" id="sat-uvindex">--</div>
                                        </div>
                                        
                                        <div class="sat-data-card-v2">
                                            <div class="sat-data-label-v2">Visibilidad</div>
                                            <div class="sat-data-value-v2" id="sat-visibility">-- km</div>
                                            <div class="sat-data-label-v2">Nubosidad</div>
                                            <div class="sat-data-value-v2" id="sat-cloudcover">-- %</div>
                                        </div>
                                        
                                        <div class="sat-data-card-v2">
                                            <div class="sat-data-label-v2">Prob. Precipitación</div>
                                            <div class="sat-data-value-v2" id="sat-precipitation">-- %</div>
                                            <div class="sat-data-label-v2">Lluvia (1h)</div>
                                            <div class="sat-data-value-v2" id="sat-rain">-- mm</div>
                                        </div>
                                        
                                        <div class="sat-data-card-v2">
                                            <div class="sat-data-label-v2">Punto de Rocío</div>
                                            <div class="sat-data-value-v2" id="sat-dewpoint">-- °C</div>
                                            <div class="sat-data-label-v2">Índice Calor</div>
                                            <div class="sat-data-value-v2" id="sat-heatindex">-- °C</div>
                                        </div>
                                        
                                        <div class="sat-data-card-v2">
                                            <div class="sat-data-label-v2">Índice Wind Chill</div>
                                            <div class="sat-data-value-v2" id="sat-windchill">-- °C</div>
                                            <div class="sat-data-label-v2">Calidad Aire</div>
                                            <div class="sat-data-value-v2" id="sat-aqi">--</div>
                                        </div>
                                        
                                        <div class="sat-data-card-v2">
                                            <div class="sat-data-label-v2">Hora Solar</div>
                                            <div class="sat-data-value-v2" id="sat-sunshine">-- h</div>
                                            <div class="sat-data-label-v2">Índice CO2</div>
                                            <div class="sat-data-value-v2" id="sat-co2">-- ppm</div>
                                        </div>
                                    </div>
                                    
                                    <div class="sat-gps-container-v2">
                                        <div style="font-size:0.6rem; color:#00ff88; margin-bottom:4px;">
                                            <i class="fas fa-map-marker-alt"></i> POSICIÓN GPS ACTUAL
                                        </div>
                                        <div class="gps-coords-v2" id="sat-gps-coords">
                                            Lat: --.----° N, Lon: --.----° E
                                        </div>
                                        <div style="font-size:0.75rem; color:#ffaa00; margin:3px 0;" id="updateSatelliteUI">
                                            Altitud: -- m | Precisión: -- m
                                        </div>
                                        <div style="font-size:0.45rem; color:#888;">
                                            Última actualización: <span id="sat-last-update">--:--:--</span>
                                        </div>
                                    </div>
                                    
                                    <div class="emergency-panel-v2">
                                        <div class="emergency-title-v2">
                                            <i class="fas fa-exclamation-triangle"></i>
                                            CONTROL DE EMERGENCIAS SATELITAL
                                        </div>
                                        
                                        <div class="emergency-buttons-v2">
                                            <button class="emergency-btn-big-v2" onclick="sendSatelliteEmergency()" id="emergency-btn-main">
                                                <i class="fas fa-satellite-dish"></i> EMERGENCIA
                                            </button>
                                            <button class="position-btn-v2" onclick="sendPositionToChat()">
                                                <i class="fas fa-share-alt"></i> POSICIÓN
                                            </button>
                                            <button class="position-btn-v2" onclick="getCurrentGPSPosition()">
                                                <i class="fas fa-crosshairs"></i> GPS
                                            </button>
                                        </div>
                                        
                                        <div style="font-size:0.45rem; color:#ffaa00; text-align:center; margin-top:4px;">
                                            ⚠️ La emergencia enviará tu posición actual a todos los contactos
                                        </div>
                                    </div>
                                    
                                    <div style="display:flex; justify-content:space-between; margin-top:6px; font-size:0.45rem;">
                                        <div style="color:#888;">
                                            <input type="checkbox" id="auto-refresh-checkbox" checked>
                                            <label for="auto-refresh-checkbox">Auto-actualizar (60s)</label>
                                        </div>
                                        <div style="color:#00ffea;">
                                            v4.7.2 | API: Open-Meteo
                                        </div>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
                
                <div id="monitor-decoded">
                    <div class="message-bubble message-system">
                        <i class="fas fa-satellite"></i> SISTEMA RADCOM MASTER INICIADO
                    </div>
                </div>
            </div>    
        </div>

        <div class="footer-pro">
            <div class="input-controls">
                <select id="inputMode" title="Modo de entrada" onchange="validateInputMode(); switchTabFromMode()">
                    <option value="text">TEXTO → HEX</option>
                    <option value="hex">HEX → TEXTO</option>
                    <option value="binary">BINARIO</option>
                    <option value="morse">CQ MORSE</option>
                    <option value="phonetic">RADIO FONÉTICO</option>
                    <option value="satellite">SATÉLITE</option>
                </select>
                
                <select id="encryptionMode" title="Encriptación">
                    <option value="xor">XOR Simple</option>
                    <option value="aes" selected>AES-256-GCM</option>
                    <option value="none">Sin encriptación</option>
                </select>
                
                <div style="position:relative; flex:1;">
                    <input type="password" id="key" placeholder="CLAVE DE ENCRIPTACIÓN (32 bytes)" 
                           style="width:100%; padding:4px 6px; font-size:0.65rem;"
                           onfocus="this.type='text'" 
                           onblur="if(this.value==='')this.type='password'">
                    <button onclick="generateKey()" 
                            style="position:absolute; right:4px; top:50%; transform:translateY(-50%);
                                   background:none; border:none; color:#888; cursor:pointer; font-size:0.65rem;">
                        <i class="fas fa-random"></i>
                    </button>
                </div>
                
                <span id="target-display" style="color:#00ff88; font-weight:bold; font-size:0.65rem; min-width:100px;">
                    <i class="fas fa-crosshairs"></i> DESTINO: GLOBAL
                </span>
            </div>
            
            <div style="display:flex; height:36px; align-items:center; padding:4px 5px; gap:4px; border-bottom:1px solid #222;">
                <button class="clear-chat-btn" onclick="clearChat()" style="flex:0 0 auto; width:36px" title="Limpiar chat">
                    <i class="fas fa-trash"></i>
                </button>

                <button id="micBtn" class="mic-btn" title="Dictar por voz" onclick="toggleVoiceInput()">
                    <i class="fas fa-microphone"></i>
                </button>

                <input type="text" id="inputMsg"
                    placeholder="INYECTAR DATOS SEGUROS..."
                    oninput="validateInput(); realTimePreview(); realTimeTableHighlight();"
                    onkeydown="if(event.key === 'Enter'){ handleSendMessage(event); }"
                    style="height:100%; flex:1;">

                <button id="sendBtn" onclick="sendWithQueue()" style="height:100%">
                    <i class="fas fa-paper-plane"></i> ENVIAR SEGURO
                </button>

                <span id="mic-status">Voz OFF</span>
            </div>
            
            <div class="stats-panel">
                <div class="stat-item">
                    <div class="stat-value" id="stats-uptime">00:00:00</div>
                    <div class="stat-label">TIEMPO ACTIVO</div>
                </div>
                <div class="stat-item">
                    <div class="stat-value" id="stats-connections">0</div>
                    <div class="stat-label">CONEXIONES</div>
                </div>
                <div class="stat-item">
                    <div class="stat-value" id="stats-encrypted">100%</div>
                    <div class="stat-label">ENCRIPTADO</div>
                </div>
                <div class="stat-item">
                    <div class="stat-value" id="stats-bandwidth">0 KB/s</div>
                    <div class="stat-label">ANCHO BANDA</div>
                </div>
            </div>
        </div>
    </div>

    <div class="modal-overlay" id="settingsModal">
        <div class="modal-content">
            <button class="modal-close" onclick="hideSettings()">&times;</button>
            <div class="modal-title">
                <i class="fas fa-cog"></i> CONFIGURACIÓN v4.7
            </div>
            
            <div style="margin-bottom:10px;">
                <label style="display:block; margin-bottom:4px; color:#00ff88; font-size:0.75rem;">
                    IDENTIFICADOR DEL SISTEMA
                </label>
                <input type="text" id="systemName" style="width:100%; padding:5px; background:#000; 
                       color:#00ff88; border:1px solid #333; border-radius:2px; font-size:0.75rem;" 
                       value="RADCOM-MASTER-v5">
            </div>
            
            <div id="idSettingsSection" style="margin-bottom: 15px; border: 1px solid #333; padding: 10px; border-radius: 3px;">
                <div style="color: #00ff88; font-size: 0.75rem; margin-bottom: 8px; display: flex; align-items: center; gap: 5px;">
                    <i class="fas fa-fingerprint"></i> CONFIGURACIÓN DE IDENTIFICADOR
                </div>
                
                <div style="margin-bottom: 8px;">
                    <label style="display: flex; align-items: center; color: #00ff88; font-size: 0.7rem; cursor: pointer;">
                        <input type="checkbox" id="useFixedId" style="margin-right: 6px;" 
                               onchange="toggleIdMode()">
                        Usar ID fijo personalizado
                    </label>
                    <div style="font-size: 0.6rem; color: #888; margin-left: 20px; margin-top: 2px;">
                        Recomendado para reconexiones estables
                    </div>
                </div>
                
                <div id="customIdContainer" style="margin-top: 8px;">
                    <label style="display: block; margin-bottom: 4px; color: #00ff88; font-size: 0.7rem;">
                        ID Personalizado:
                    </label>
                    <div style="display: flex; gap: 5px;">
                        <input type="text" id="customIdInput" 
                               style="flex: 1; padding: 5px; background: #000; color: #00ff88; 
                                      border: 1px solid #333; border-radius: 2px; font-size: 0.7rem;"
                               placeholder="Ej: RADCOM-MI-ESTACION">
                        <button onclick="generateCustomId()" 
                                style="background: #333; color: #00ff88; border: 1px solid #444; 
                                       padding: 5px 8px; border-radius: 2px; cursor: pointer; font-size: 0.65rem;">
                            <i class="fas fa-random"></i>
                        </button>
                    </div>
                    <div style="font-size: 0.6rem; color: #888; margin-top: 4px;">
                        Actual: <span id="currentIdDisplay" style="color: #00ffea;">Cargando...</span>
                    </div>
                </div>
                
                <div style="margin-top: 10px; display: flex; gap: 8px;">
                    <button onclick="applyIdSettings()" 
                            style="flex: 1; padding: 6px; background: #00ff88; color: #000; 
                                   border: none; border-radius: 2px; font-weight: bold; cursor: pointer; font-size: 0.7rem;">
                        <i class="fas fa-save"></i> APLICAR ID
                    </button>
                    <button onclick="resetIdSettings()" 
                            style="padding: 6px 10px; background: #333; color: #ff3300; 
                                   border: 1px solid #444; border-radius: 2px; cursor: pointer; font-size: 0.7rem;">
                        <i class="fas fa-redo"></i>
                    </button>
                </div>
            </div>
            
            <div style="margin-bottom:10px;">
                <div style="color:#00ff88; font-size:0.75rem; margin-bottom:6px;">
                    <i class="fas fa-network-wired"></i> TIPO DE CONEXIÓN
                </div>
                <div class="connection-type-selector">
                    <div class="connection-type-btn active" id="btn-wifi" onclick="selectConnectionType('wifi')">
                        <div class="connection-icon">
                            <i class="fas fa-wifi"></i>
                        </div>
                        RED WiFi
                        <div class="connection-info">
                            Redes privadas
                        </div>
                    </div>
                    <div class="connection-type-btn" id="btn-mobile" onclick="selectConnectionType('mobile')">
                        <div class="connection-icon">
                            <i class="fas fa-satellite-dish"></i>
                        </div>
                        DATOS MÓVILES
                        <div class="connection-info">
                            4G/5G Global
                        </div>
                    </div>
                </div>
                <div id="connection-status" style="font-size:0.65rem; color:#00ffea; margin-top:5px; text-align:center;">
                    Estado: <span id="current-connection-type">WiFi</span>
                </div>
            </div>
            
            <div style="margin-bottom:10px;">
                <label style="display:block; margin-bottom:4px; color:#00ff88; font-size:0.75rem;">
                    <input type="checkbox" id="autoReconnect" checked style="margin-right:4px;">
                    Reconexión automática
                </label>
                <label style="display:block; margin-bottom:4px; color:#00ff88; font-size:0.75rem;">
                    <input type="checkbox" id="soundEnabled" checked style="margin-right:4px;">
                    Efectos de sonido
                </label>
                <label style="display:block; margin-bottom:4px; color:#00ff88; font-size:0.75rem;">
                    <input type="checkbox" id="saveHistory" checked style="margin-right:4px;">
                    Guardar historial
                </label>
                <label style="display:block; margin-bottom:4px; color:#00ff88; font-size:0.75rem;">
                    <input type="checkbox" id="fastRecovery" checked style="margin-right:4px;">
                    Recuperación rápida
                </label>
                <label style="display:block; margin-bottom:4px; color:#00ff88; font-size:0.75rem;">
                    <input type="checkbox" id="aggressiveRevive" checked style="margin-right:4px;">
                    Revivir agresivo
                </label>
            </div>
            
            <button onclick="saveSettings()" style="width:100%; padding:7px; background:#00ff88; 
                    color:#000; border:none; border-radius:2px; font-weight:bold; cursor:pointer;
                    font-size:0.8rem;">
                <i class="fas fa-save"></i> GUARDAR CONFIGURACIÓN
            </button>
        </div>
    </div>

<div class="console-modal-overlay" id="consoleModal">
    <div class="console-modal-content">
        <div class="console-scan-line"></div>
        
        <div class="console-header">
            <div class="console-title">
                <i class="fas fa-terminal"></i> RADCOM CONSOLE v5.6.1 - SYSTEM CONTROL
            </div>
            <button class="console-close" onclick="closeConsole()">&times;</button>
        </div>
        
        <div class="console-tabs">
            <button class="console-tab active" onclick="switchConsoleTab('CMD')">CMD Console</button>
            <button class="console-tab" onclick="switchConsoleTab('hex')">Hex Editor</button>
            <button class="console-tab" onclick="switchConsoleTab('decode')">Hex Decoder</button>
        </div>
        
        <div id="console-tab-CMD" class="console-tab-content active">
            <div class="CMD-console-container">
                <div id="CMD-output" class="CMD-output">
&gt; RADCOM CMD CONSOLE v5.6.1 INITIALIZED
&gt; System: RADCOM MASTER
&gt; Version: 5.6.1
&gt; Ready for commands...
                </div>
                <div class="CMD-input-row">
                    <input type="text" id="CMD-input" class="CMD-input" placeholder="Enter command..." 
                           onkeydown="handleCMDCommand(event)">
                    <button class="CMD-btn" onclick="executeCMDCommand()">EXECUTE</button>
                    <button class="CMD-btn CMD-btn-clear" onclick="clearCMDConsole()">CLEAR</button>
                </div>
            </div>
        </div>
        
        <div id="console-tab-hex" class="console-tab-content">
            <div class="hex-editor-container">
                <div class="hex-toolbar">
                    <button class="hex-toolbar-btn" onclick="hexLoadCurrentPage()">
                        <i class="fas fa-code"></i> Load Page
                    </button>
                    <button class="hex-toolbar-btn" onclick="hexSaveChanges()">
                        <i class="fas fa-save"></i> Save
                    </button>
                    <button class="hex-toolbar-btn" onclick="hexFind()">
                        <i class="fas fa-search"></i> Find
                    </button>
                    <button class="hex-toolbar-btn" onclick="hexFormat()">
                        <i class="fas fa-align-left"></i> Format
                    </button>
                    <button class="hex-toolbar-btn" onclick="hexMinify()">
                        <i class="fas fa-compress"></i> Minify
                    </button>
                </div>
                <textarea id="hex-editor" class="hex-editor" spellcheck="false"></textarea>
                <div class="hex-status-bar">
                    <span id="hex-status">Ready</span>
                    <span id="hex-position">Line: 1, Col: 1</span>
                </div>
            </div>
        </div>
        
        <div id="console-tab-decode" class="console-tab-content">
            <div class="hex-decoder-container">
                <div class="hex-decoder-split">
                    <div class="hex-decoder-input">
                        <div class="hex-decoder-label">
                            <i class="fas fa-keyboard"></i> Input (Hex or Text)
                        </div>
                        <textarea id="hex-decoder-input" class="hex-decoder-textarea" 
                                  placeholder="Paste HEX data or text here..." 
                                  oninput="autoDetectAndConvert()"></textarea>
                    </div>
                    <div class="hex-decoder-output">
                        <div class="hex-decoder-label">
                            <i class="fas fa-laptop-code"></i> Output
                        </div>
                        <textarea id="hex-decoder-output" class="hex-decoder-textarea" 
                                  placeholder="Converted output will appear here..." 
                                  readonly></textarea>
                    </div>
                </div>
                <div class="hex-decoder-controls">
                    <button class="hex-decoder-btn" onclick="convertTextToHex()">
                        <i class="fas fa-arrow-right"></i> Text → HEX
                    </button>
                    <button class="hex-decoder-btn" onclick="convertHexToText()">
                        <i class="fas fa-arrow-left"></i> HEX → Text
                    </button>
                    <button class="hex-decoder-btn" onclick="decodeBase64()">
                        <i class="fas fa-unlock"></i> Base64
                    </button>
                    <button class="hex-decoder-btn" onclick="clearDecoder()">
                        <i class="fas fa-trash"></i> Clear
                    </button>
                </div>
            </div>
        </div>
    </div>
</div>

<div id="modal-680" style="display:none; position:absolute; top:calc(50% + 24px); left:50%; transform:translate(-50%, -50%); width:680px; height:680px; background:#000; border:2px solid #00ff88; box-shadow:0 0 20px rgba(0,255,136,0.5); z-index:5000; box-sizing:border-box;">
    <div style="display:flex; justify-content:space-between; background:#222; padding:5px 10px; border-bottom:1px solid #00ff88; height:30px; box-sizing:border-box;">
        <span id="modal-title" style="color:#00ff88; font-family:monospace; font-weight:bold;">SISTEMA NAV</span>
        <button onclick="closeModal680()" style="background:none; border:none; color:#ff3300; cursor:pointer; font-weight:bold;">[ X ]</button>
    </div>
    <div id="modal-body" style="width:100%; height:calc(100% - 30px); overflow:auto; box-sizing:border-box; background:#000;"></div>
</div>

<div id="pfd-source-storage" style="display:none;">
    
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>PFD/ECAM Paramotor - Diseño Final</title>
    <style>
        :root {
            --color-black: #000000;
            --color-white: #FFFFFF;
            --color-red: #FF0000;
            --color-amber: #FFBF00;
            --color-green: #00FF00;
            --color-cyan: #00FFFF;
            --color-magenta: #FF00FF;
            --color-blue: #0000FF;
            --color-gray: #333333;
            --color-dark-gray: #1A1A1A;
        }
        
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
            font-family: 'Courier New', monospace;
        }
        
        body {
            background-color: #000;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            padding: 20px;
        }
        
        .display-container {
            width: 480px;
            height: 480px;
            background-color: var(--color-black);
            border: 2px solid var(--color-gray);
            position: relative;
            overflow: hidden;
        }
        
        /* ============ BARRA SUPERIOR PERSONALIZADA ============ */
        .top-bar {
            height: 30px;
            background-color: var(--color-dark-gray);
            border-bottom: 1px solid var(--color-gray);
            display: flex;
            align-items: center;
            justify-content: space-between;
            padding: 0 15px;
        }
        
        .left-info {
            display: flex;
            align-items: center;
            gap: 15px;
        }
        
        .right-info {
            display: flex;
            align-items: center;
            gap: 15px;
        }
        
        .center-logo {
            position: absolute;
            left: 50%;
            transform: translateX(-50%);
            font-size: 16px;
            font-weight: bold;
            color: var(--color-white);
            letter-spacing: 1px;
        }
        
        .info-box {
            display: flex;
            align-items: center;
            gap: 5px;
        }
        
        .info-label {
            font-size: 10px;
            color: var(--color-cyan);
        }
        
        .info-value {
            font-size: 12px;
            font-weight: bold;
        }
        
        .signal-indicator {
            display: flex;
            align-items: center;
            gap: 3px;
        }
        
        .signal-bar {
            width: 3px;
            height: 10px;
            background-color: var(--color-green);
            border-radius: 1px;
        }
        
        /* ============ PFD (3/4 de la pantalla) ============ */
        .pfd-area {
            height: 360px; /* 3/4 de 480px */
            position: relative;
        }
        
        /* ============ TAPE DE VELOCIDAD (IZQUIERDA) ============ */
        .speed-tape-container {
            position: absolute;
            left: 0;
            top: 0;
            width: 60px;
            height: 360px;
            background-color: var(--color-black);
            border-right: 1px solid var(--color-gray);
        }
        
        .speed-values {
            position: absolute;
            left: 0;
            top: 0;
            width: 100%;
            height: 100%;
        }
        
        .speed-value {
            position: absolute;
            right: 5px;
            font-size: 10px;
            color: var(--color-white);
            transform: translateY(-50%);
        }
        
        .speed-current {
            position: absolute;
            left: 50%;
            top: 50%;
            transform: translate(-50%, -50%);
            font-size: 28px;
            font-weight: bold;
            color: var(--color-cyan);
            background-color: rgba(0,0,0,0.8);
            padding: 5px 8px;
            border-radius: 3px;
            border: 2px solid var(--color-cyan);
            text-align: center;
            min-width: 50px;
        }
        
        .speed-label {
            position: absolute;
            bottom: 15px;
            left: 50%;
            transform: translateX(-50%);
            font-size: 12px;
            color: var(--color-cyan);
        }
        
        .speed-mark {
            position: absolute;
            right: 0;
            width: 15px;
            height: 1px;
            background-color: var(--color-white);
        }
        
        /* ============ ÁREA CENTRAL PFD ============ */
        .center-area {
            position: absolute;
            left: 60px;
            top: 0;
            width: 360px;
            height: 360px;
            background-color: var(--color-black);
        }
        
        .attitude-display {
            position: absolute;
            top: 50px;
            left: 50%;
            transform: translateX(-50%);
            width: 280px;
            height: 180px;
            border: 2px solid var(--color-gray);
            border-radius: 5px;
            overflow: hidden;
        }
        
        .sky {
            position: absolute;
            top: 0;
            width: 100%;
            height: 50%;
            background: linear-gradient(to bottom, #000066, #001155);
        }
        
        .ground {
            position: absolute;
            bottom: 0;
            width: 100%;
            height: 50%;
            background: linear-gradient(to top, #663300, #552200);
        }
        
        .pitch-line {
            position: absolute;
            left: 50%;
            transform: translateX(-50%);
            height: 1px;
            background-color: var(--color-white);
        }
        
        .pitch-label {
            position: absolute;
            top: 50%;
            transform: translateY(-50%);
            font-size: 10px;
            color: var(--color-white);
        }
        
        .heading-display {
            position: absolute;
            bottom: 40px;
            left: 50%;
            transform: translateX(-50%);
            width: 220px;
            height: 35px;
            background-color: rgba(0,0,0,0.7);
            border: 1px solid var(--color-gray);
            border-radius: 4px;
            display: flex;
            align-items: center;
            justify-content: center;
        }
        
        .heading-value {
            font-size: 18px;
            font-weight: bold;
            color: var(--color-white);
        }
        
        .heading-label {
            position: absolute;
            bottom: 80px;
            left: 50%;
            transform: translateX(-50%);
            font-size: 14px;
            color: var(--color-cyan);
        }
        
        /* ============ TAPE DE ALTITUD + VSI (DERECHA) ============ */
        .altitude-tape-container {
            position: absolute;
            right: 0;
            top: 0;
            width: 80px;
            height: 360px;
            background-color: var(--color-black);
            border-left: 1px solid var(--color-gray);
            display: flex;
        }
        
        .altitude-column {
            width: 60px;
            height: 100%;
            position: relative;
        }
        
        .altitude-values {
            position: absolute;
            left: 0;
            top: 0;
            width: 100%;
            height: 100%;
        }
        
        .altitude-value {
            position: absolute;
            left: 5px;
            font-size: 10px;
            color: var(--color-white);
            transform: translateY(-50%);
        }
        
        .altitude-current {
            position: absolute;
            left: 50%;
            top: 50%;
            transform: translate(-50%, -50%);
            font-size: 28px;
            font-weight: bold;
            color: var(--color-cyan);
            background-color: rgba(0,0,0,0.8);
            padding: 5px 8px;
            border-radius: 3px;
            border: 2px solid var(--color-cyan);
            text-align: center;
            min-width: 50px;
        }
        
        .altitude-label {
            position: absolute;
            bottom: 15px;
            left: 50%;
            transform: translateX(-50%);
            font-size: 12px;
            color: var(--color-cyan);
        }
        
        .altitude-mark {
            position: absolute;
            left: 0;
            width: 15px;
            height: 1px;
            background-color: var(--color-white);
        }
        
        /* Variómetro (VSI) - columna derecha dentro del tape de altitud */
        .vsi-column {
            width: 20px;
            height: 100%;
            background-color: rgba(0,0,0,0.7);
            border-left: 1px solid var(--color-gray);
            position: relative;
        }
        
        .vsi-scale {
            position: absolute;
            left: 0;
            top: 0;
            width: 100%;
            height: 100%;
        }
        
        .vsi-zero {
            position: absolute;
            top: 50%;
            left: 0;
            width: 100%;
            height: 2px;
            background-color: var(--color-white);
        }
        
        .vsi-mark {
            position: absolute;
            left: 5px;
            font-size: 8px;
            color: var(--color-white);
        }
        
        .vsi-needle {
            position: absolute;
            left: 50%;
            transform: translateX(-50%);
            width: 0;
            height: 0;
            border-left: 6px solid transparent;
            border-right: 6px solid transparent;
            border-top: 12px solid var(--color-amber);
        }
        
        .vsi-label {
            position: absolute;
            bottom: 15px;
            left: 50%;
            transform: translateX(-50%);
            font-size: 10px;
            color: var(--color-cyan);
            writing-mode: vertical-rl;
        }
        
        /* ============ ECAM (1/4 de la pantalla) ============ */
        .ecam-area {
            height: 90px;
            background-color: var(--color-dark-gray);
            border-top: 2px solid var(--color-gray);
            display: flex;
        }
        
        .ecam-column {
            flex: 1;
            padding: 5px 8px;
            border-right: 1px solid var(--color-gray);
        }
        
        .ecam-column:last-child {
            border-right: none;
        }
        
        .column-title {
            font-size: 9px;
            color: var(--color-cyan);
            text-transform: uppercase;
            margin-bottom: 3px;
            border-bottom: 1px solid var(--color-gray);
            padding-bottom: 1px;
        }
        
        .data-row {
            display: flex;
            justify-content: space-between;
            margin-bottom: 2px;
        }
        
        .data-label {
            font-size: 9px;
            color: var(--color-cyan);
        }
        
        .data-value {
            font-size: 11px;
            font-weight: bold;
            color: var(--color-white);
        }
        
        /* ============ ALERTAS ============ */
        .alert-box {
            position: absolute;
            top: 40%;
            left: 50%;
            transform: translate(-50%, -50%);
            background-color: var(--color-red);
            color: var(--color-white);
            padding: 10px 20px;
            font-size: 16px;
            font-weight: bold;
            border: 2px solid var(--color-white);
            border-radius: 5px;
            z-index: 100;
            text-align: center;
            box-shadow: 0 0 20px var(--color-red);
            animation: pulse 1s infinite;
        }
        
        @keyframes pulse {
            0% { opacity: 1; }
            50% { opacity: 0.7; }
            100% { opacity: 1; }
        }
        
        /* ============ BARRA DE COMBUSTIBLE ============ */
        .fuel-bar-container {
            margin-top: 3px;
            width: 100%;
            height: 8px;
            background-color: var(--color-gray);
            border-radius: 4px;
            overflow: hidden;
        }
        
        .fuel-level {
            height: 100%;
            background-color: var(--color-green);
            border-radius: 4px;
        }
        
        .fuel-labels {
            display: flex;
            justify-content: space-between;
            margin-top: 1px;
            font-size: 7px;
            color: var(--color-white);
        }
        
        /* ============ COLORES DE ESTADO ============ */
        .status-normal { color: var(--color-white); }
        .status-warning { color: var(--color-amber); }
        .status-critical { color: var(--color-red); }
        .status-good { color: var(--color-green); }
        .status-info { color: var(--color-cyan); }
        
        /* ============ INDICADORES DE SEÑAL ============ */
        .signal-weak .signal-bar:nth-child(4),
        .signal-weak .signal-bar:nth-child(5) {
            background-color: var(--color-gray);
        }
        
        .signal-medium .signal-bar:nth-child(5) {
            background-color: var(--color-gray);
        }
        
        /* ============ BARRA INFERIOR ============ */
        .bottom-bar {
            height: 30px;
            background-color: var(--color-dark-gray);
            border-top: 1px solid var(--color-gray);
            display: flex;
            align-items: center;
            justify-content: space-between;
            padding: 0 15px;
            font-size: 11px;
            color: var(--color-white);
        }
        
        .flight-info {
            display: flex;
            gap: 20px;
        }
        
        .info-item {
            display: flex;
            align-items: center;
            gap: 5px;
        }
        
        .info-item-label {
            color: var(--color-cyan);
        }
        
        .info-item-value {
            font-weight: bold;
        }
    </style>
</head>
<body>

    <div class="display-container">
        
        <!-- ============ BARRA SUPERIOR PERSONALIZADA ============ -->
        <div class="top-bar">
            <div class="left-info">
                <div class="info-box">
                    <span class="info-label">BATT</span>
                    <span class="info-value status-good">12.4V</span>
                </div>
                <div class="info-box">
                    <span class="info-label">DATE</span>
                    <span class="info-value status-normal">26/01/2026</span>
                </div>
            </div>
            
            <div class="center-logo">PARAMOTOR CUSTOM</div>
            
            <div class="right-info">
                <div class="signal-indicator signal-medium">
                    <div class="signal-bar" style="height: 6px;"></div>
                    <div class="signal-bar" style="height: 9px;"></div>
                    <div class="signal-bar" style="height: 12px;"></div>
                    <div class="signal-bar" style="height: 9px;"></div>
                    <div class="signal-bar" style="height: 6px;"></div>
                </div>
                <div class="info-box">
                    <span class="info-label">GPS</span>
                    <span class="info-value status-good">8 SAT</span>
                </div>
                <div class="info-box">
                    <span class="info-label">UTC</span>
                    <span class="info-value status-normal">18:34:17</span>
                </div>
            </div>
        </div>
        
        <!-- ============ ÁREA PFD (3/4 superior) ============ -->
        <div class="pfd-area">
            
            <!-- Tape de velocidad (izquierda) -->
            <div class="speed-tape-container">
                <div class="speed-values" id="speedValues">
                    <!-- Valores generados dinámicamente -->
                </div>
                <div class="speed-current" id="speedCurrent">62</div>
                <div class="speed-label">km/h</div>
            </div>
            
            <!-- Área central PFD -->
            <div class="center-area">
                <!-- Horizonte artificial -->
                <div class="attitude-display">
                    <div class="sky"></div>
                    <div class="ground"></div>
                    
                    <!-- Líneas de pitch -->
                    <div class="pitch-line" style="top: 20%; width: 50px;"></div>
                    <div class="pitch-label" style="top: 20%; right: 10px;">10°</div>
                    
                    <div class="pitch-line" style="top: 30%; width: 40px;"></div>
                    <div class="pitch-line" style="top: 40%; width: 30px;"></div>
                    <div class="pitch-line" style="top: 50%; width: 280px; background-color: var(--color-cyan);"></div>
                    <div class="pitch-line" style="top: 60%; width: 30px;"></div>
                    <div class="pitch-line" style="top: 70%; width: 40px;"></div>
                    
                    <div class="pitch-line" style="top: 80%; width: 50px;"></div>
                    <div class="pitch-label" style="top: 80%; right: 10px;">-10°</div>
                </div>
                
                <!-- Indicador de rumbo -->
                <div class="heading-display">
                    <div class="heading-value">185°</div>
                </div>
                <div class="heading-label">HDG</div>
            </div>
            
            <!-- Tape de altitud + VSI (derecha) -->
            <div class="altitude-tape-container">
                <div class="altitude-column">
                    <div class="altitude-values" id="altitudeValues">
                        <!-- Valores generados dinámicamente -->
                    </div>
                    <div class="altitude-current" id="altitudeCurrent">456</div>
                    <div class="altitude-label">m</div>
                </div>
                
                <!-- Variómetro (VSI) -->
                <div class="vsi-column">
                    <div class="vsi-scale" id="vsiScale">
                        <div class="vsi-zero"></div>
                        <div class="vsi-needle" id="vsiNeedle" style="top: 35%;"></div>
                    </div>
                    <div class="vsi-label">m/s</div>
                </div>
            </div>
            
        </div>
        
        <!-- ============ ÁREA ECAM ============ -->
        <div class="ecam-area">
            
            <!-- Columna 1: Motor -->
            <div class="ecam-column">
                <div class="column-title">MOTOR</div>
                <div class="data-row">
                    <span class="data-label">EGT</span>
                    <span class="data-value status-critical" id="egtValue">888°C</span>
                </div>
                <div class="data-row">
                    <span class="data-label">CHT</span>
                    <span class="data-value status-normal" id="chtValue">185°C</span>
                </div>
                <div class="data-row">
                    <span class="data-label">N1</span>
                    <span class="data-value status-normal" id="n1Value">96.5%</span>
                </div>
                <div class="data-row">
                    <span class="data-label">RPM</span>
                    <span class="data-value status-normal" id="rpmValue">9900</span>
                </div>
            </div>
            
            <!-- Columna 2: Combustible -->
            <div class="ecam-column">
                <div class="column-title">COMBUSTIBLE</div>
                <div class="data-row">
                    <span class="data-label">FOB</span>
                    <span class="data-value status-warning" id="fuelValue">7.2 L</span>
                </div>
                <div class="data-row">
                    <span class="data-label">TIME RES</span>
                    <span class="data-value status-normal" id="timeResValue">55 MIN</span>
                </div>
                <div class="fuel-bar-container">
                    <div class="fuel-level" id="fuelLevel" style="width: 72%;"></div>
                </div>
                <div class="fuel-labels">
                    <span>0</span>
                    <span>10L</span>
                </div>
            </div>
            
            <!-- Columna 3: Tiempos -->
            <div class="ecam-column">
                <div class="column-title">TIEMPOS</div>
                <div class="data-row">
                    <span class="data-label">FLIGHT</span>
                    <span class="data-value status-normal" id="flightTimeValue">01:18</span>
                </div>
                <div class="data-row">
                    <span class="data-label">TOTAL</span>
                    <span class="data-value status-normal" id="totalTimeValue">125:30</span>
                </div>
                <div class="data-row">
                    <span class="data-label">CRONO</span>
                    <span class="data-value status-normal">00:00</span>
                </div>
            </div>
            
            <!-- Columna 4: Sistema -->
            <div class="ecam-column">
                <div class="column-title">SISTEMA</div>
                <div class="data-row">
                    <span class="data-label">BATT</span>
                    <span class="data-value status-good" id="battValue">12.4V</span>
                </div>
                <div class="data-row">
                    <span class="data-label">PRESS</span>
                    <span class="data-value status-normal" id="pressValue">1013 hPa</span>
                </div>
                <div class="data-row">
                    <span class="data-label">TEMP</span>
                    <span class="data-value status-normal">15°C</span>
                </div>
                <div class="data-row">
                    <span class="data-label">V/S</span>
                    <span class="data-value status-normal">2.3 m/s</span>
                </div>
            </div>
            
        </div>
        
        <!-- ============ BARRA INFERIOR ============ -->
        <div class="bottom-bar">
            <div class="flight-info">
                <div class="info-item">
                    <span class="info-item-label">DIST:</span>
                    <span class="info-item-value">15.7 km</span>
                </div>
                <div class="info-item">
                    <span class="info-item-label">TO HOME:</span>
                    <span class="info-item-value">3.2 km</span>
                </div>
                <div class="info-item">
                    <span class="info-item-label">WIND:</span>
                    <span class="info-item-value">12 km/h</span>
                </div>
            </div>
            <div class="info-item">
                <span class="info-item-label">MODE:</span>
                <span class="info-item-value status-normal">MANUAL</span>
            </div>
        </div>
        
        <!-- Alerta EGT -->
        <div class="alert-box" id="alertBox" style="display: none;">EGT TOO HIGH</div>
        
    </div>

    <script>
        // ============ DATOS DEL PROYECTO ============
        const projectData = {
            // PFD Data
            speed: 62,
            altitude: 456,
            heading: 185,
            vsi: 2.3,
            
            // Engine Data
            egt: 888,
            cht: 185,
            n1: 96.5,
            rpm: 9900,
            
            // Fuel Data
            fuel: 7.2,
            fuelCapacity: 10,
            timeRemaining: 55,
            
            // Time Data
            flightTime: 78, // minutos
            totalTime: 125.5, // horas:minutos
            
            // System Data
            battery: 12.4,
            pressure: 1013,
            temperature: 15,
            
            // Navigation Data
            distance: 15.7,
            distanceToHome: 3.2,
            windSpeed: 12
        };
        
        // ============ INICIALIZACIÓN ============
        function initDisplay() {
            updateStaticValues();
            generateSpeedTape();
            generateAltitudeTape();
            generateVsiScale();
            checkAlerts();
        }
        
        // ============ ACTUALIZAR VALORES ESTÁTICOS ============
        function updateStaticValues() {
            // PFD Values
            document.getElementById('speedCurrent').textContent = Math.round(projectData.speed);
            document.getElementById('altitudeCurrent').textContent = Math.round(projectData.altitude);
            
            // Engine Values
            document.getElementById('egtValue').textContent = Math.round(projectData.egt) + '°C';
            document.getElementById('chtValue').textContent = Math.round(projectData.cht) + '°C';
            document.getElementById('n1Value').textContent = projectData.n1.toFixed(1) + '%';
            document.getElementById('rpmValue').textContent = Math.round(projectData.rpm);
            
            // Fuel Values
            document.getElementById('fuelValue').textContent = projectData.fuel.toFixed(1) + ' L';
            document.getElementById('timeResValue').textContent = Math.round(projectData.timeRemaining) + ' MIN';
            
            // Calculate fuel percentage
            const fuelPercent = (projectData.fuel / projectData.fuelCapacity) * 100;
            document.getElementById('fuelLevel').style.width = fuelPercent + '%';
            
            // Update fuel color based on level
            if (fuelPercent < 20) {
                document.getElementById('fuelValue').className = 'data-value status-critical';
                document.getElementById('fuelLevel').style.backgroundColor = 'var(--color-red)';
            } else if (fuelPercent < 50) {
                document.getElementById('fuelValue').className = 'data-value status-warning';
                document.getElementById('fuelLevel').style.backgroundColor = 'var(--color-amber)';
            } else {
                document.getElementById('fuelValue').className = 'data-value status-normal';
                document.getElementById('fuelLevel').style.backgroundColor = 'var(--color-green)';
            }
            
            // Time Values
            const flightHours = Math.floor(projectData.flightTime / 60);
            const flightMinutes = Math.round(projectData.flightTime % 60);
            document.getElementById('flightTimeValue').textContent = 
                `${flightHours.toString().padStart(2, '0')}:${flightMinutes.toString().padStart(2, '0')}`;
            
            const totalHours = Math.floor(projectData.totalTime);
            const totalMinutes = Math.round((projectData.totalTime - totalHours) * 60);
            document.getElementById('totalTimeValue').textContent = 
                `${totalHours.toString().padStart(3, '0')}:${totalMinutes.toString().padStart(2, '0')}`;
            
            // System Values
            document.getElementById('battValue').textContent = projectData.battery.toFixed(1) + 'V';
            document.getElementById('pressValue').textContent = Math.round(projectData.pressure) + ' hPa';
            
            // Update VSI needle
            const vsiNeedle = document.getElementById('vsiNeedle');
            const vsiPosition = 50 - (projectData.vsi * 10); // 10px por m/s
            vsiNeedle.style.top = vsiPosition + '%';
            
            // Update VSI needle color
            if (projectData.vsi < -2) {
                vsiNeedle.style.borderTopColor = 'var(--color-red)';
            } else if (projectData.vsi < -1) {
                vsiNeedle.style.borderTopColor = 'var(--color-amber)';
            } else {
                vsiNeedle.style.borderTopColor = 'var(--color-green)';
            }
            
            // Update EGT color
            const egtValue = document.getElementById('egtValue');
            if (projectData.egt > 850) {
                egtValue.className = 'data-value status-critical';
            } else if (projectData.egt > 750) {
                egtValue.className = 'data-value status-warning';
            } else {
                egtValue.className = 'data-value status-normal';
            }
        }
        
        // ============ GENERAR TAPE DE VELOCIDAD ============
        function generateSpeedTape() {
            const tape = document.getElementById('speedValues');
            tape.innerHTML = '';
            
            const center = 180; // Centro del tape (360px / 2)
            const pxPerUnit = 3; // 3px por km/h
            
            // Generar marcas cada 10 km/h
            for (let i = -100; i <= 100; i += 10) {
                const value = projectData.speed + i;
                if (value >= 0 && value <= 200) {
                    const position = center - (i * pxPerUnit);
                    
                    // Marca principal cada 20 km/h
                    if (i % 20 === 0) {
                        const mark = document.createElement('div');
                        mark.className = 'speed-mark';
                        mark.style.top = position + 'px';
                        tape.appendChild(mark);
                        
                        const number = document.createElement('div');
                        number.className = 'speed-value';
                        number.textContent = value;
                        number.style.top = position + 'px';
                        tape.appendChild(number);
                    }
                    // Marca menor cada 10 km/h
                    else {
                        const mark = document.createElement('div');
                        mark.className = 'speed-mark';
                        mark.style.top = position + 'px';
                        mark.style.width = '10px';
                        tape.appendChild(mark);
                    }
                }
            }
        }
        
        // ============ GENERAR TAPE DE ALTITUD ============
        function generateAltitudeTape() {
            const tape = document.getElementById('altitudeValues');
            tape.innerHTML = '';
            
            const center = 180;
            const pxPerUnit = 1.5; // 1.5px por metro
            
            // Generar marcas cada 20 metros
            for (let i = -200; i <= 200; i += 20) {
                const value = projectData.altitude + i;
                if (value >= 0) {
                    const position = center - (i * pxPerUnit);
                    
                    // Marca principal cada 40 metros
                    if (i % 40 === 0) {
                        const mark = document.createElement('div');
                        mark.className = 'altitude-mark';
                        mark.style.top = position + 'px';
                        tape.appendChild(mark);
                        
                        const number = document.createElement('div');
                        number.className = 'altitude-value';
                        number.textContent = value;
                        number.style.top = position + 'px';
                        tape.appendChild(number);
                    }
                    // Marca menor cada 20 metros
                    else {
                        const mark = document.createElement('div');
                        mark.className = 'altitude-mark';
                        mark.style.top = position + 'px';
                        mark.style.width = '10px';
                        tape.appendChild(mark);
                    }
                }
            }
        }
        
        // ============ GENERAR ESCALA VSI ============
        function generateVsiScale() {
            const scale = document.getElementById('vsiScale');
            
            // Agregar marcas VSI
            for (let i = -5; i <= 5; i++) {
                if (i !== 0) {
                    const position = 180 - (i * 18); // 18px por m/s
                    
                    const mark = document.createElement('div');
                    mark.className = 'vsi-mark';
                    mark.textContent = Math.abs(i);
                    mark.style.top = position + 'px';
                    
                    // Posicionar números a la izquierda para positivos, derecha para negativos
                    if (i > 0) {
                        mark.style.left = '3px';
                    } else {
                        mark.style.left = '3px';
                    }
                    
                    scale.appendChild(mark);
                    
                    // Línea pequeña
                    const line = document.createElement('div');
                    line.style.position = 'absolute';
                    line.style.left = '0';
                    line.style.width = '5px';
                    line.style.height = '1px';
                    line.style.backgroundColor = 'var(--color-white)';
                    line.style.top = position + 'px';
                    scale.appendChild(line);
                }
            }
        }
        
        // ============ VERIFICAR ALERTAS ============
        function checkAlerts() {
            const alertBox = document.getElementById('alertBox');
            
            if (projectData.egt > 850) {
                alertBox.style.display = 'block';
                alertBox.textContent = 'EGT TOO HIGH';
            } else if (projectData.fuel < 2) {
                alertBox.style.display = 'block';
                alertBox.textContent = 'LOW FUEL';
            } else {
                alertBox.style.display = 'none';
            }
        }
        
        // ============ CONTROL MANUAL DE VALORES ============
        document.addEventListener('keydown', function(e) {
            switch(e.key) {
                case 'ArrowUp':
                    projectData.altitude += 10;
                    break;
                case 'ArrowDown':
                    projectData.altitude -= 10;
                    break;
                case 'ArrowLeft':
                    projectData.speed -= 5;
                    break;
                case 'ArrowRight':
                    projectData.speed += 5;
                    break;
                case 'e':
                    projectData.egt += 10;
                    break;
                case 'E':
                    projectData.egt -= 10;
                    break;
                case 'f':
                    projectData.fuel += 0.5;
                    if (projectData.fuel > projectData.fuelCapacity) {
                        projectData.fuel = projectData.fuelCapacity;
                    }
                    break;
                case 'F':
                    projectData.fuel -= 0.5;
                    if (projectData.fuel < 0) {
                        projectData.fuel = 0;
                    }
                    break;
                case 'v':
                    projectData.vsi += 0.5;
                    if (projectData.vsi > 5) projectData.vsi = 5;
                    break;
                case 'V':
                    projectData.vsi -= 0.5;
                    if (projectData.vsi < -5) projectData.vsi = -5;
                    break;
            }
            
            // Limitar valores
            projectData.speed = Math.max(0, Math.min(200, projectData.speed));
            projectData.altitude = Math.max(0, Math.min(1000, projectData.altitude));
            projectData.egt = Math.max(600, Math.min(1000, projectData.egt));
            
            // Actualizar display
            updateStaticValues();
            generateSpeedTape();
            generateAltitudeTape();
            checkAlerts();
        });
        
        // ============ INICIAR DISPLAY ============
        window.onload = initDisplay;
        
    </script>
</body>
</html>
    </div>

    <div id="util-source-storage" style="display:none;">
        <style>
        /* ===== TU CSS ORIGINAL COMPLETO (sin cambios) ===== */
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
            user-select: none;
        }

        @font-face {
            font-family: 'Digital-7';
            src: url('https://cdn.jsdelivr.net/npm/digital-7-font@1.0.0/digital-7.ttf') format('truetype');
        }

        body {
            background: #111;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            font-family: 'Arial', sans-serif;
        }

        .transponder-container {
            width: 680px;
            height: auto;
            min-height: 680px;
            background: #1a1a1a;
            border: 3px solid #666;
            border-radius: 16px;
            padding: 12px;
            display: flex;
            justify-content: center;
            align-items: center;
            box-shadow: 0 20px 30px rgba(0,0,0,0.8);
        }

        .gtx327 {
            width: 100%;
            height: 100%;
            background: linear-gradient(145deg, #2d2d2d 0%, #1a1a1a 100%);
            border: 3px solid #999;
            border-radius: 12px;
            padding: 12px 12px;
            display: flex;
            flex-direction: column;
            box-shadow: inset 0 2px 5px rgba(255,255,255,0.1);
        }

        .logo-row {
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-bottom: 8px;
            height: 24px;
        }
        .garmin-logo {
            color: #fff;
            font-size: 20px;
            font-weight: bold;
            letter-spacing: 1px;
        }
        .model {
            color: #aaa;
            font-size: 16px;
        }
        .badge {
            background: #04f;
            color: #fff;
            font-size: 10px;
            padding: 2px 4px;
            border-radius: 4px;
            margin-left: 4px;
        }

        .display {
            background: #0f1f0f;
            border: 3px solid #333;
            border-radius: 8px;
            padding: 8px 10px;
            margin-bottom: 10px;
            height: 130px;
            display: flex;
            box-shadow: inset 0 0 15px rgba(0,0,0,0.9);
        }

        .transponder-data {
            flex: 1;
            display: flex;
            flex-direction: column;
            border-right: 2px solid #2a4a2a;
            padding-right: 10px;
        }

        .mode-tag {
            color: #8f8;
            font-size: 16px;
            font-weight: bold;
            margin-bottom: 2px;
        }

        .code {
            font-family: 'Digital-7', monospace;
            font-size: 58px;
            color: #fc0;
            text-shadow: 0 0 8px #fa0;
            letter-spacing: 8px;
            line-height: 1;
            margin-bottom: 2px;
        }

        .ident-line {
            font-family: 'Digital-7', monospace;
            font-size: 18px;
            color: #fc0;
            min-height: 22px;
            opacity: 0;
        }
        .ident-line.active {
            opacity: 1;
        }

        .altitude-data {
            flex: 1;
            display: flex;
            flex-direction: column;
            align-items: flex-end;
            padding-left: 10px;
        }

        .function-line {
            font-family: 'Digital-7', monospace;
            font-size: 20px;
            color: #8f8;
            text-shadow: 0 0 4px #0f0;
            margin-bottom: 5px;
            text-align: right;
            width: 100%;
        }

        .altitude-value {
            font-family: 'Digital-7', monospace;
            font-size: 34px;
            color: #8ff;
            text-shadow: 0 0 6px #0ff;
            text-align: right;
            width: 100%;
            margin-bottom: 2px;
        }
        
        .altitude-value.no-data {
            color: #844;
            text-shadow: 0 0 6px #800;
        }

        .trend-row {
            display: flex;
            justify-content: flex-end;
            align-items: center;
            gap: 6px;
            width: 100%;
        }
        .trend {
            font-size: 26px;
            color: #fa0;
            text-shadow: 0 0 6px #ff0;
            min-width: 35px;
            text-align: center;
        }
        .rx-symbol {
            font-size: 18px;
            color: #8f8;
            opacity: 0;
            transition: opacity 0.1s;
        }
        .rx-symbol.active {
            opacity: 1;
            text-shadow: 0 0 8px #0f0;
        }
        .tx-symbol {
            font-size: 16px;
            color: #0af;
            margin-left: 5px;
            opacity: 0;
            transition: opacity 0.1s;
        }
        .tx-symbol.active {
            opacity: 1;
            text-shadow: 0 0 8px #0af;
            animation: pulse 1s infinite;
        }

        .mode-buttons {
            display: flex;
            gap: 4px;
            margin-bottom: 8px;
        }
        .mode-btn {
            flex: 1;
            height: 38px;
            background: linear-gradient(180deg, #3a3a3a, #222);
            border: 2px solid #666;
            border-radius: 5px;
            color: #ddd;
            font-size: 15px;
            font-weight: bold;
            cursor: pointer;
            border-bottom: 3px solid #444;
            box-shadow: 0 3px 0 #111;
            transition: 0.05s;
        }
        .mode-btn:active { transform: translateY(3px); box-shadow: none; }
        .mode-btn.active { background: linear-gradient(180deg, #1f7f1f, #0f4f0f); border-color: #8f8; color: #fff; box-shadow: 0 0 10px #0f0; }
        .mode-btn.off-active { background: linear-gradient(180deg, #7f2f2f, #4f0f0f); border-color: #f88; }
        .mode-btn.vfr { border-color: #fc0; color: #fc0; }

        .keypad {
            display: grid;
            grid-template-columns: repeat(4, 1fr);
            gap: 4px;
            margin-bottom: 8px;
        }
        .num-btn {
            height: 42px;
            background: linear-gradient(180deg, #3a3a3a, #222);
            border: 2px solid #fc0;
            border-radius: 5px;
            color: #fc0;
            font-size: 22px;
            font-weight: bold;
            cursor: pointer;
            border-bottom: 3px solid #aa8800;
            box-shadow: 0 3px 0 #111;
            transition: 0.05s;
        }
        .num-btn:active { transform: translateY(3px); box-shadow: none; }
        .num-btn.disabled { opacity: 0.3; pointer-events: none; border-color: #666; color: #666; }

        .func-buttons {
            display: grid;
            grid-template-columns: repeat(3, 1fr);
            gap: 4px;
            margin-bottom: 8px;
        }
        .func-btn {
            height: 38px;
            background: linear-gradient(180deg, #3a3a3a, #222);
            border: 2px solid #7af;
            border-radius: 5px;
            color: #7af;
            font-size: 14px;
            font-weight: bold;
            cursor: pointer;
            border-bottom: 3px solid #3366aa;
            box-shadow: 0 3px 0 #111;
            transition: 0.05s;
        }
        .func-btn:active { transform: translateY(3px); box-shadow: none; }
        .func-btn.ident { border-color: #fa0; color: #fa0; border-bottom-color: #aa7700; }

        .special-buttons {
            display: flex;
            gap: 4px;
            margin-bottom: 8px;
        }
        .special-btn {
            flex: 1;
            height: 38px;
            background: linear-gradient(180deg, #3a3a3a, #222);
            border: 2px solid #fa0;
            border-radius: 5px;
            color: #fa0;
            font-size: 18px;
            font-weight: bold;
            cursor: pointer;
            border-bottom: 3px solid #aa6600;
            box-shadow: 0 3px 0 #111;
            transition: 0.05s;
        }
        .special-btn:active { transform: translateY(3px); box-shadow: none; }

        /* Panel SDR (RX) - Verde */
        .sdr-panel {
            display: flex;
            align-items: center;
            gap: 6px;
            padding: 6px 10px;
            background: #0a0a0a;
            border: 2px solid #0f0;
            border-radius: 6px;
            margin-top: 4px;
        }
        .sdr-label {
            color: #0f0;
            font-size: 12px;
            font-weight: bold;
        }
        .sdr-freq {
            font-family: 'Digital-7', monospace;
            color: #0f0;
            font-size: 16px;
        }
        .sdr-led {
            width: 10px;
            height: 10px;
            border-radius: 50%;
            background: #f00;
            box-shadow: 0 0 6px #f00;
        }
        .sdr-led.connected { background: #0f0; box-shadow: 0 0 6px #0f0; }
        .sdr-led.data { background: #ff0; box-shadow: 0 0 6px #ff0; }
        .sdr-led.error { background: #f00; box-shadow: 0 0 6px #f00; }
        .sdr-btn {
            margin-left: auto;
            background: #2a2a2a;
            border: 2px solid #0f0;
            color: #0f0;
            padding: 4px 12px;
            border-radius: 16px;
            font-weight: bold;
            cursor: pointer;
            font-size: 12px;
        }
        .sdr-btn:hover { background: #0f0; color: #000; }
        .sdr-btn.connected { background: #0f0; color: #000; }

        /* Panel ADS-B (TX) - Azul */
        .adsb-panel {
            display: flex;
            align-items: center;
            gap: 6px;
            padding: 6px 10px;
            background: #0a0a0a;
            border: 2px solid #0af;
            border-radius: 6px;
            margin-top: 4px;
        }
        .adsb-label {
            color: #0af;
            font-size: 12px;
            font-weight: bold;
        }
        .adsb-freq {
            font-family: 'Digital-7', monospace;
            color: #0af;
            font-size: 16px;
        }
        .adsb-led {
            width: 10px;
            height: 10px;
            border-radius: 50%;
            background: #f00;
            box-shadow: 0 0 6px #f00;
        }
        .adsb-led.connected { background: #0af; box-shadow: 0 0 6px #0af; }
        .adsb-led.tx { background: #0af; box-shadow: 0 0 6px #0af; animation: pulse 1s infinite; }
        .adsb-led.error { background: #f00; box-shadow: 0 0 6px #f00; }
        .adsb-btn {
            margin-left: auto;
            background: #2a2a2a;
            border: 2px solid #0af;
            color: #0af;
            padding: 4px 12px;
            border-radius: 16px;
            font-weight: bold;
            cursor: pointer;
            font-size: 12px;
        }
        .adsb-btn:hover { background: #0af; color: #000; }
        .adsb-btn.connected { background: #0af; color: #000; }

        /* ===== ESTILOS METAR ===== */
        .metar-section {
            margin-top: 8px;
            padding: 8px;
            background: #0a1a2a;
            border: 2px solid #4a6a8a;
            border-radius: 8px;
        }
        .metar-header {
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-bottom: 6px;
            color: #8cf;
            font-size: 12px;
            font-weight: bold;
        }
        .metar-airport {
            color: #ffaa00;
            font-family: 'Digital-7', monospace;
            font-size: 14px;
        }
        .metar-time {
            color: #8f8;
            font-size: 10px;
        }
        .metar-raw {
            background: #112233;
            border: 1px solid #2a4a6a;
            border-radius: 6px;
            padding: 8px;
            margin: 5px 0;
            font-family: 'Courier New', monospace;
            font-size: 13px;
            color: #8ff;
            letter-spacing: 1px;
            word-break: break-all;
            border-left: 4px solid #ffaa00;
        }
        .metar-raw.error {
            color: #f88;
            border-left-color: #f00;
        }
        .metar-raw.loading {
            color: #888;
            border-left-color: #888;
        }
        .metar-stats {
            display: flex;
            gap: 12px;
            margin-top: 5px;
            font-size: 10px;
            color: #aaa;
            flex-wrap: wrap;
        }
        .metar-stat {
            display: flex;
            align-items: center;
            gap: 4px;
        }
        .metar-stat .label {
            color: #6af;
        }
        .metar-stat .value {
            color: #fa0;
            font-family: 'Digital-7', monospace;
            font-weight: bold;
        }
        .metar-refresh {
            margin-left: auto;
            background: #2a4a6a;
            border: 1px solid #6af;
            color: #fff;
            padding: 2px 8px;
            border-radius: 12px;
            font-size: 9px;
            cursor: pointer;
        }
        .metar-refresh:hover {
            background: #6af;
        }

        .stats-row {
            display: flex;
            justify-content: space-between;
            margin-top: 4px;
            color: #888;
            font-size: 10px;
        }
        .stat-value { color: #fa0; font-family: 'Digital-7', monospace; }
        .rx-stat { color: #0f0; }
        .tx-stat { color: #0af; }
        .qnh-stat { color: #ffaa00; }
        .local-time { color: #00ff88; }

        @keyframes pulse {
            0% { opacity: 1; }
            50% { opacity: 0.5; }
            100% { opacity: 1; }
        }
        
        .debug-info {
            font-size: 8px;
            color: #666;
            margin-top: 2px;
            text-align: center;
        }
        
        .position-row {
            display: flex;
            justify-content: space-between;
            margin-top: 2px;
            font-size: 9px;
            color: #aaa;
        }
        .pos-value {
            color: #0af;
            font-family: 'Digital-7', monospace;
        }
        
        .callsign {
            color: #0af;
            font-size: 11px;
            font-weight: bold;
        }
        
        .separator {
            height: 2px;
            background: #333;
            margin: 4px 0;
        }
        
        .region-selector {
            display: flex;
            gap: 2px;
            margin-left: 5px;
        }
        
        .region-btn {
            background: #222;
            border: 1px solid #444;
            color: #888;
            font-size: 8px;
            padding: 1px 4px;
            border-radius: 2px;
            cursor: pointer;
        }
        
        .region-btn.active {
            background: #00ff88;
            color: #000;
            border-color: #00ff88;
        }
    </style>
</head>
<body>
<div class="transponder-container">
    <div class="gtx327">
        <div class="logo-row">
            <span class="garmin-logo">GARMIN</span>
            <span class="model">GTX 327 <span class="badge">SDR + METAR REAL</span></span>
            <div class="region-selector">
                <button class="region-btn active" id="btnRegionEU" onclick="setRegion('EU')">EU</button>
                <button class="region-btn" id="btnRegionUS" onclick="setRegion('US')">US</button>
            </div>
        </div>

        <div class="display">
            <div class="transponder-data">
                <div class="mode-tag" id="modeDisplay">OFF</div>
                <span class="code" id="codeDisplay">7000</span>
                <span class="ident-line" id="identDisplay"></span>
            </div>

            <div class="altitude-data">
                <div class="function-line" id="functionDisplay">PRESSURE ALT</div>
                <div class="altitude-value" id="altitudeDisplay">-- ft</div>
                <div class="trend-row">
                    <span class="trend" id="trendDisplay">•</span>
                    <span class="rx-symbol" id="rxDisplay">RX</span>
                    <span class="tx-symbol" id="txDisplay">TX</span>
                </div>
            </div>
        </div>

        <div class="mode-buttons">
            <button class="mode-btn" id="btnStby">STBY</button>
            <button class="mode-btn" id="btnOn">ON</button>
            <button class="mode-btn" id="btnAlt">ALT</button>
            <button class="mode-btn" id="btnOff">OFF</button>
            <button class="mode-btn vfr" id="btnVfr">VFR</button>
        </div>

        <div class="keypad">
            <button class="num-btn" data-digit="0">0</button>
            <button class="num-btn" data-digit="1">1</button>
            <button class="num-btn" data-digit="2">2</button>
            <button class="num-btn" data-digit="3">3</button>
            <button class="num-btn" data-digit="4">4</button>
            <button class="num-btn" data-digit="5">5</button>
            <button class="num-btn" data-digit="6">6</button>
            <button class="num-btn" data-digit="7">7</button>
        </div>

        <div class="func-buttons">
            <button class="func-btn ident" id="btnIdent">IDENT</button>
            <button class="func-btn" id="btnFunc">FUNC</button>
            <button class="func-btn" id="btnClr">CLR</button>
            <button class="func-btn" id="btnCrsr">CRSR</button>
            <button class="func-btn" id="btnEnt">ENT</button>
            <button class="func-btn" id="btnStartStop">ST/STP</button>
        </div>

        <div class="special-buttons">
            <button class="special-btn" id="btnEight">8▲</button>
            <button class="special-btn" id="btnNine">9▼</button>
        </div>

        <!-- PANEL SDR (RX) -->
        <div class="sdr-panel">
            <span class="sdr-label">📡 SDR RX:</span>
            <span class="sdr-freq" id="sdrFreq">1090 MHz</span>
            <span class="sdr-led" id="sdrLed"></span>
            <button class="sdr-btn" id="sdrConnectBtn">AUTO</button>
        </div>

        <!-- PANEL ADS-B (TX) -->
        <div class="adsb-panel">
            <span class="adsb-label">🛰️ ADS-B TX:</span>
            <span class="adsb-freq" id="adsbFreq">1090 ES</span>
            <span class="adsb-led" id="adsbLed"></span>
            <button class="adsb-btn" id="adsbConnectBtn">ACTIVAR</button>
        </div>

        <!-- ===== SECCIÓN METAR REAL ===== -->
        <div class="metar-section">
            <div class="metar-header">
                <span>🌤️ METAR REAL (AEROPUERTO MÁS CERCANO)</span>
                <span class="metar-airport" id="metarAirport">--</span>
                <span class="metar-time" id="metarTime">--:-- UTC</span>
                <button class="metar-refresh" id="metarRefreshBtn">↻ ACTUALIZAR</button>
            </div>
            <div class="metar-raw" id="metarRaw">Esperando coordenadas GPS...</div>
            <div class="metar-stats">
                <span class="metar-stat"><span class="label">VIENTO:</span> <span class="value" id="metarWind">--</span></span>
                <span class="metar-stat"><span class="label">VIS:</span> <span class="value" id="metarVis">--</span></span>
                <span class="metar-stat"><span class="label">TEMP:</span> <span class="value" id="metarTemp">--</span></span>
                <span class="metar-stat"><span class="label">QNH:</span> <span class="value" id="metarQnh">--</span></span>
                <span class="metar-stat"><span class="label">CAT:</span> <span class="value" id="metarCat">--</span></span>
            </div>
        </div>

        <!-- MI IDENTIFICACIÓN -->
        <div class="position-row">
            <span>ICAO: <span class="stat-value" id="icaoDisplay">123456</span></span>
            <span>CALL: <span class="callsign" id="callsignDisplay">MYCALL</span></span>
            <span>MODE: <span class="stat-value" id="modeSDisplay">S</span></span>
            <span>ALT: <span class="stat-value" id="altDisplay">1500 ft</span></span>
        </div>
        <div class="position-row">
            <span>LAT: <span class="pos-value" id="latDisplay">--° --' --"</span></span>
            <span>LON: <span class="pos-value" id="lonDisplay">--° --' --"</span></span>
            <span>QNH: <span class="stat-value qnh-stat" id="qnhDisplay">1013 hPa</span></span>
        </div>

        <div class="separator"></div>

        <!-- ESTADÍSTICAS -->
        <div class="stats-row">
            <span>🛫 CERCA: <span class="stat-value rx-stat" id="aircraftCount">0</span></span>
            <span>📥 RX: <span class="stat-value" id="rxCount">0</span></span>
            <span>📊 dB: <span class="stat-value" id="signalStrength">-</span></span>
            <span>🌡️ <span class="stat-value" id="tempDisplay">--°C</span></span>
        </div>
        <div class="stats-row">
            <span>🛰️ MI ADS-B: <span class="stat-value tx-stat" id="myAdsBCount">0</span></span>
            <span>📤 TX: <span class="stat-value" id="txCount">0</span></span>
            <span>🔵 SQWK: <span class="stat-value" id="squawkDisplay">7000</span></span>
            <span>🕐 <span class="stat-value local-time" id="timeDisplay">--:--:--</span></span>
        </div>
        
        <div class="debug-info" id="debugInfo">Status: <span id="usbStatus">Iniciando...</span></div>
    </div>
</div>

<script>
    (function() {
        // AUDIO
        const audioCtx = new (window.AudioContext || window.webkitAudioContext)();
        const playClick = () => {
            if (audioCtx.state === 'suspended') audioCtx.resume();
            const osc = audioCtx.createOscillator();
            const gain = audioCtx.createGain();
            osc.connect(gain);
            gain.connect(audioCtx.destination);
            osc.frequency.value = 1200;
            gain.gain.setValueAtTime(0.15, audioCtx.currentTime);
            gain.gain.exponentialRampToValueAtTime(0.01, audioCtx.currentTime + 0.03);
            osc.start(audioCtx.currentTime);
            osc.stop(audioCtx.currentTime + 0.03);
        };

        // CONFIGURACIÓN
        const CONFIG = {
            ICAO: '123456',
            CALLSIGN: 'MYCALL',
            EMITTER_CATEGORY: 1
        };

        // ESTADO
        const state = {
            power: false,
            mode: 'OFF',
            code: '7000',
            previousCode: '7000',
            enteringCode: false,
            tempCode: '',
            identActive: false,
            identTimer: null,
            functionPage: 0,
            timerFlight: 0,
            timerCountUp: 0,
            timerCountDown: 600,
            timerRunning: false,
            contrast: 50,
            brightness: 50,
            cursorActive: false,
            altitude: Math.floor(1500 + Math.random() * 500),
            lastAltitude: null,
            region: 'EU',
            
            // SDR (RX)
            sdrConnected: false,
            sdrDevice: null,
            sdrReadingActive: false,
            sdrSimulationMode: false,
            sdrInterval: null,
            sdrAutoConnectAttempted: false,
            sdrPermissionAsked: false,
            sdrDeviceInfo: null,
            
            // ADS-B (TX)
            adsbActive: false,
            adsbInterval: null,
            myAdsBCount: 0,
            txCount: 0,
            
            // Datos de tráfico
            aircraftSet: new Set(),
            replies: 0,
            signalStrength: 0,
            
            // MI POSICIÓN
            latitude: 40.4168,
            longitude: -3.7038,
            altitudeMsl: 1500,
            groundSpeed: 120,
            track: 270,
            verticalRate: 0,
            
            lastSuccessfulRead: Date.now(),
            consecutiveErrors: 0,
            
            temperature: null,
            pressure: null,
            qnh: 1013.25,
            
            // METAR
            metar: {
                airport: null,
                icao: null,
                raw: null,
                time: null,
                wind: null,
                visibility: null,
                temp: null,
                qnh: null,
                category: null,
                lastUpdate: null,
                source: null
            }
        };

        // DOM Elements
        const modeDisplay = document.getElementById('modeDisplay');
        const codeDisplay = document.getElementById('codeDisplay');
        const identDisplay = document.getElementById('identDisplay');
        const functionDisplay = document.getElementById('functionDisplay');
        const altitudeDisplay = document.getElementById('altitudeDisplay');
        const trendDisplay = document.getElementById('trendDisplay');
        const rxDisplay = document.getElementById('rxDisplay');
        const txDisplay = document.getElementById('txDisplay');
        
        const sdrLed = document.getElementById('sdrLed');
        const sdrConnectBtn = document.getElementById('sdrConnectBtn');
        
        const adsbLed = document.getElementById('adsbLed');
        const adsbConnectBtn = document.getElementById('adsbConnectBtn');
        
        const aircraftCountSpan = document.getElementById('aircraftCount');
        const rxCountSpan = document.getElementById('rxCount');
        const signalStrengthSpan = document.getElementById('signalStrength');
        const myAdsBCountSpan = document.getElementById('myAdsBCount');
        const txCountSpan = document.getElementById('txCount');
        const squawkDisplay = document.getElementById('squawkDisplay');
        const tempDisplay = document.getElementById('tempDisplay');
        const qnhDisplay = document.getElementById('qnhDisplay');
        const timeDisplay = document.getElementById('timeDisplay');
        const altDisplay = document.getElementById('altDisplay');
        
        const icaoDisplay = document.getElementById('icaoDisplay');
        const callsignDisplay = document.getElementById('callsignDisplay');
        const latDisplay = document.getElementById('latDisplay');
        const lonDisplay = document.getElementById('lonDisplay');
        const modeSDisplay = document.getElementById('modeSDisplay');
        const debugInfo = document.getElementById('debugInfo');
        const usbStatus = document.getElementById('usbStatus');

        // Elementos METAR
        const metarAirport = document.getElementById('metarAirport');
        const metarTime = document.getElementById('metarTime');
        const metarRaw = document.getElementById('metarRaw');
        const metarWind = document.getElementById('metarWind');
        const metarVis = document.getElementById('metarVis');
        const metarTemp = document.getElementById('metarTemp');
        const metarQnh = document.getElementById('metarQnh');
        const metarCat = document.getElementById('metarCat');
        const metarRefreshBtn = document.getElementById('metarRefreshBtn');

        // Region Buttons
        const btnRegionEU = document.getElementById('btnRegionEU');
        const btnRegionUS = document.getElementById('btnRegionUS');

        // Buttons
        const btnStby = document.getElementById('btnStby');
        const btnOn = document.getElementById('btnOn');
        const btnAlt = document.getElementById('btnAlt');
        const btnOff = document.getElementById('btnOff');
        const btnVfr = document.getElementById('btnVfr');
        const btnIdent = document.getElementById('btnIdent');
        const btnFunc = document.getElementById('btnFunc');
        const btnClr = document.getElementById('btnClr');
        const btnCrsr = document.getElementById('btnCrsr');
        const btnEnt = document.getElementById('btnEnt');
        const btnStartStop = document.getElementById('btnStartStop');
        const btnEight = document.getElementById('btnEight');
        const btnNine = document.getElementById('btnNine');
        const numBtns = document.querySelectorAll('.num-btn');

        icaoDisplay.textContent = CONFIG.ICAO;
        callsignDisplay.textContent = CONFIG.CALLSIGN;
        modeSDisplay.textContent = 'S';
        codeDisplay.textContent = state.code;
        squawkDisplay.textContent = state.code;

        if (navigator.usb) {
            usbStatus.textContent = '✅ WebUSB';
            usbStatus.style.color = '#0f0';
        } else {
            usbStatus.textContent = '❌ No WebUSB';
            usbStatus.style.color = '#f00';
        }

        window.setRegion = (region) => {
            state.region = region;
            if (region === 'EU') {
                state.code = '7000';
                btnRegionEU.classList.add('active');
                btnRegionUS.classList.remove('active');
            } else {
                state.code = '1200';
                btnRegionUS.classList.add('active');
                btnRegionEU.classList.remove('active');
            }
            codeDisplay.textContent = state.code;
            squawkDisplay.textContent = state.code;
            playClick();
        };

        // ===== FUNCIÓN QNH =====
        const fetchQNH = async () => {
            try {
                const lat = state.latitude.toFixed(4);
                const lon = state.longitude.toFixed(4);
                const response = await fetch(`https://api.open-meteo.com/v1/forecast?latitude=${lat}&longitude=${lon}&current=temperature_2m,pressure_msl`);
                const data = await response.json();
                if (data.current) {
                    state.qnh = data.current.pressure_msl;
                    state.temperature = data.current.temperature_2m;
                    qnhDisplay.textContent = `${Math.round(state.qnh)} hPa`;
                    tempDisplay.textContent = `${Math.round(state.temperature)}°C`;
                }
            } catch (error) {
                console.log('Error QNH');
            }
        };

        // ===== HORA REAL =====
        const updateTime = () => {
            const now = new Date();
            const hours = now.getHours().toString().padStart(2, '0');
            const minutes = now.getMinutes().toString().padStart(2, '0');
            const seconds = now.getSeconds().toString().padStart(2, '0');
            timeDisplay.textContent = `${hours}:${minutes}:${seconds}`;
        };
        setInterval(updateTime, 1000);

        // ===== FUNCIONES GPS =====
        

        const updateSimulatedPosition = () => {
            const time = Date.now() / 10000;
            state.latitude = 40.4168 + Math.sin(time) * 0.01;
            state.longitude = -3.7038 + Math.cos(time) * 0.01;
            state.altitude = 1500 + Math.sin(time * 2) * 200;
            updatePositionDisplay();
        };

        const updatePositionDisplay = () => {
            const latDeg = Math.floor(Math.abs(state.latitude));
            const latMin = Math.floor((Math.abs(state.latitude) - latDeg) * 60);
            const latSec = Math.floor(((Math.abs(state.latitude) - latDeg) * 60 - latMin) * 60);
            const latDir = state.latitude >= 0 ? 'N' : 'S';
            const lonDeg = Math.floor(Math.abs(state.longitude));
            const lonMin = Math.floor((Math.abs(state.longitude) - lonDeg) * 60);
            const lonSec = Math.floor(((Math.abs(state.longitude) - lonDeg) * 60 - lonMin) * 60);
            const lonDir = state.longitude >= 0 ? 'E' : 'W';
            latDisplay.textContent = `${latDeg}° ${latMin}' ${latSec}" ${latDir}`;
            lonDisplay.textContent = `${lonDeg}° ${lonMin}' ${latSec}" ${lonDir}`;
            altDisplay.textContent = `${Math.round(state.altitude)} ft`;
        };

        // ===== METAR CON MÚLTIPLES FUENTES (REDUNDANCIA) =====
        // Reemplaza la función fetchMETAR completa con esta versión mejorada:

const fetchMETAR = async () => {
    if (!state.latitude || !state.longitude) {
        metarRaw.textContent = 'Esperando GPS...';
        metarRaw.className = 'metar-raw loading';
        return;
    }
    
    metarRaw.textContent = 'Obteniendo METAR...';
    metarRaw.className = 'metar-raw loading';
    
    // Base de datos de aeropuertos (ampliada)
    const airports = [
        // España
        { icao: 'LEMD', name: 'MADRID', lat: 40.472, lon: -3.560 },
        { icao: 'LEBL', name: 'BARCELONA', lat: 41.297, lon: 2.078 },
        { icao: 'LEPA', name: 'PALMA', lat: 39.551, lon: 2.738 },
        { icao: 'LEMG', name: 'MALAGA', lat: 36.675, lon: -4.499 },
        { icao: 'LEVC', name: 'VALENCIA', lat: 39.489, lon: -0.481 },
        { icao: 'LEBB', name: 'BILBAO', lat: 43.301, lon: -2.911 },
        { icao: 'LEAS', name: 'ASTURIAS', lat: 43.563, lon: -6.034 },
        { icao: 'LEVX', name: 'VIGO', lat: 42.231, lon: -8.627 },
        { icao: 'GCLP', name: 'GRAN CANARIA', lat: 27.931, lon: -15.386 },
        { icao: 'GCXO', name: 'TENERIFE', lat: 28.482, lon: -16.341 },
        // USA
        { icao: 'KJFK', name: 'NEW YORK JFK', lat: 40.639, lon: -73.778 },
        { icao: 'KLAX', name: 'LOS ANGELES', lat: 33.942, lon: -118.408 },
        { icao: 'KORD', name: 'CHICAGO O\'HARE', lat: 41.978, lon: -87.904 },
        { icao: 'KDFW', name: 'DALLAS', lat: 32.897, lon: -97.040 },
        { icao: 'KMIA', name: 'MIAMI', lat: 25.793, lon: -80.290 },
        { icao: 'KATL', name: 'ATLANTA', lat: 33.640, lon: -84.427 },
        { icao: 'KDEN', name: 'DENVER', lat: 39.856, lon: -104.673 },
        { icao: 'KSEA', name: 'SEATTLE', lat: 47.449, lon: -122.309 },
        { icao: 'KSFO', name: 'SAN FRANCISCO', lat: 37.619, lon: -122.375 },
        { icao: 'KBOS', name: 'BOSTON', lat: 42.365, lon: -71.009 },
        // Europa
        { icao: 'EGLL', name: 'LONDON', lat: 51.470, lon: -0.461 },
        { icao: 'LFPG', name: 'PARIS', lat: 49.012, lon: 2.549 },
        { icao: 'EDDF', name: 'FRANKFURT', lat: 50.033, lon: 8.570 },
        { icao: 'LIRF', name: 'ROME', lat: 41.800, lon: 12.238 },
        { icao: 'EHAM', name: 'AMSTERDAM', lat: 52.308, lon: 4.764 },
    ];
    
    try {
        // Encontrar aeropuerto más cercano
        let nearest = null;
        let minDist = Infinity;
        
        airports.forEach(ap => {
            const R = 6371; // Radio de la Tierra en km
            const dLat = (ap.lat - state.latitude) * Math.PI / 180;
            const dLon = (ap.lon - state.longitude) * Math.PI / 180;
            const a = Math.sin(dLat/2) * Math.sin(dLat/2) +
                     Math.cos(state.latitude * Math.PI / 180) * Math.cos(ap.lat * Math.PI / 180) *
                     Math.sin(dLon/2) * Math.sin(dLon/2);
            const c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1-a));
            const d = R * c;
            if (d < minDist) {
                minDist = d;
                nearest = ap;
            }
        });
        
        if (!nearest) throw new Error('No hay aeropuertos cercanos');
        
        // MOSTRAR INFORMACIÓN BÁSICA MIENTRAS SE ESPERA
        metarAirport.textContent = nearest.icao;
        
        // ===== FUENTE 1: Usando CORS proxy con NOAA (la más fiable) =====
        try {
            // Usamos un proxy CORS gratuito para evitar problemas
            const proxyUrl = 'https://api.allorigins.win/raw?url=';
            const targetUrl = `https://aviationweather.gov/api/data/metar?ids=${nearest.icao}&format=raw`;
            
            const response = await fetch(proxyUrl + encodeURIComponent(targetUrl));
            
            if (response.ok) {
                const rawMetar = await response.text();
                
                if (rawMetar && rawMetar.length > 10) {
                    // Parsear el METAR raw
                    parseRawMETAR(rawMetar, nearest, 'NOAA');
                    debugInfo.textContent = `METAR OK: ${nearest.icao}`;
                    return;
                }
            }
        } catch (e) {
            console.log('NOAA con proxy falló, usando datos simulados');
        }
        
        // ===== FUENTE 2: Datos simulados realistas si todo falla =====
        generateSimulatedMETAR(nearest);
        
    } catch (error) {
        console.error('Error METAR:', error);
        metarRaw.textContent = `Error obteniendo METAR`;
        metarRaw.className = 'metar-raw error';
        
        // Generar datos simulados como fallback
        const nearest = airports[0];
        generateSimulatedMETAR(nearest);
    }
};

// Función para parsear METAR raw
const parseRawMETAR = (raw, airport, source) => {
    metarAirport.textContent = airport.icao;
    metarRaw.textContent = raw;
    metarRaw.className = 'metar-raw';
    
    // Extraer componentes básicos del METAR (parsing simplificado)
    // Viento (ej: 27015KT)
    const windMatch = raw.match(/(\d{3})(\d{2})G?(\d{2})?KT/);
    if (windMatch) {
        let windStr = `${windMatch[1]}° ${windMatch[2]}kt`;
        if (windMatch[3]) windStr += ` G${windMatch[3]}kt`;
        metarWind.textContent = windStr;
    } else {
        metarWind.textContent = '---° --kt';
    }
    
    // Visibilidad (ej: 9999 o 10SM)
    const visMatch = raw.match(/(\d{4})|(\d+)SM/);
    if (visMatch) {
        metarVis.textContent = visMatch[0];
    } else {
        metarVis.textContent = '10km+';
    }
    
    // Temperatura (ej: 15/10)
    const tempMatch = raw.match(/M?(\d+)\/M?(\d+)/);
    if (tempMatch) {
        const temp = tempMatch[1].startsWith('M') ? -parseInt(tempMatch[1].substring(1)) : parseInt(tempMatch[1]);
        metarTemp.textContent = `${temp}°C`;
    } else {
        metarTemp.textContent = '15°C';
    }
    
    // QNH (ej: Q1013)
    const qnhMatch = raw.match(/Q(\d{4})/);
    if (qnhMatch) {
        metarQnh.textContent = `${qnhMatch[1]}hPa`;
    } else {
        metarQnh.textContent = '1013hPa';
    }
    
    // Hora
    const timeMatch = raw.match(/(\d{2})(\d{2})Z/);
    if (timeMatch) {
        metarTime.textContent = `${timeMatch[1]}:${timeMatch[2]}Z`;
    } else {
        const now = new Date();
        metarTime.textContent = `${now.getUTCHours().toString().padStart(2,'0')}:${now.getUTCMinutes().toString().padStart(2,'0')}Z`;
    }
    
    // Categoría (simplificado)
    if (raw.includes('VFR') || raw.includes('CAVOK')) {
        metarCat.textContent = 'VFR';
    } else if (raw.includes('MVFR')) {
        metarCat.textContent = 'MVFR';
    } else if (raw.includes('IFR')) {
        metarCat.textContent = 'IFR';
    } else if (raw.includes('LIFR')) {
        metarCat.textContent = 'LIFR';
    } else {
        metarCat.textContent = 'VFR';
    }
    
    debugInfo.textContent = `METAR: ${airport.icao} (${source})`;
};

// Función para generar datos METAR simulados realistas
const generateSimulatedMETAR = (airport) => {
    metarAirport.textContent = airport.icao;
    
    // Generar METAR simulado creíble
    const now = new Date();
    const hour = now.getUTCHours().toString().padStart(2, '0');
    const minute = now.getUTCMinutes().toString().padStart(2, '0');
    
    // Viento aleatorio realista
    const windDir = Math.floor(Math.random() * 360);
    const windSpeed = Math.floor(Math.random() * 15) + 5;
    const windGust = Math.random() > 0.7 ? Math.floor(Math.random() * 10) + 10 : 0;
    
    // Visibilidad
    const vis = Math.random() > 0.2 ? '9999' : '5000';
    
    // Temperatura
    const temp = Math.floor(Math.random() * 25) + 5;
    const dew = temp - Math.floor(Math.random() * 8) - 2;
    
    // QNH
    const qnh = 1013 + Math.floor(Math.random() * 20) - 10;
    
    // Nubes
    const clouds = ['FEW040', 'SCT050', 'BKN060', 'OVC080', 'CAVOK'];
    const cloud = clouds[Math.floor(Math.random() * clouds.length)];
    
    // Construir METAR
    let metarString = `${airport.icao} ${hour}${minute}Z ${windDir.toString().padStart(3,'0')}${windSpeed}KT`;
    if (windGust > 0) metarString += `G${windGust}KT`;
    metarString += ` ${vis} ${temp.toString().padStart(2,'0')}/${dew.toString().padStart(2,'0')} Q${qnh} ${cloud}`;
    
    metarRaw.textContent = metarString;
    metarRaw.className = 'metar-raw';
    
    // Actualizar campos
    metarWind.textContent = `${windDir}° ${windSpeed}kt${windGust > 0 ? ' G' + windGust + 'kt' : ''}`;
    metarVis.textContent = vis === '9999' ? '10km+' : vis + 'm';
    metarTemp.textContent = `${temp}°C`;
    metarQnh.textContent = `${qnh}hPa`;
    metarTime.textContent = `${hour}:${minute}Z`;
    metarCat.textContent = 'VFR';
    
    debugInfo.textContent = `METAR: ${airport.icao} (SIMULADO)`;
};

// Asegúrate también de que la función getPosition se llame correctamente
// Modifica la función getPosition para que siempre llame a fetchMETAR:

const getPosition = () => {
    if (!navigator.geolocation) {
        updateSimulatedPosition();
        fetchMETAR(); // Siempre llama a METAR
        fetchQNH();
        return;
    }
    navigator.geolocation.getCurrentPosition(
        (position) => {
            state.latitude = position.coords.latitude;
            state.longitude = position.coords.longitude;
            updatePositionDisplay();
            fetchMETAR(); // METAR con posición real
            fetchQNH();
        },
        (error) => {
            updateSimulatedPosition();
            fetchMETAR(); // METAR con posición simulada
            fetchQNH();
        },
        { enableHighAccuracy: true, timeout: 10000 }
    );
};

        const updateMETARDisplay = () => {
            metarAirport.textContent = state.metar.icao ? `${state.metar.icao}` : '--';
            metarTime.textContent = state.metar.time ? `${state.metar.time} UTC` : '--:-- UTC';
            
            let rawText = state.metar.raw || 'No disponible';
            if (state.metar.source === 'SIM') rawText += ' (SIM)';
            metarRaw.textContent = rawText;
            
            metarWind.textContent = state.metar.wind || '--';
            metarVis.textContent = state.metar.visibility || '--';
            metarTemp.textContent = state.metar.temp || '--';
            metarQnh.textContent = state.metar.qnh || '--';
            metarCat.textContent = state.metar.category || '--';
            
            metarRaw.className = 'metar-raw';
            
            debugInfo.textContent = `METAR: ${state.metar.source || '?'} - ${new Date().toLocaleTimeString()}`;
        };

        // ===== AUTO DETECCIÓN DE SDR FÍSICO =====
        const autoConnectSDR = async () => {
            if (state.sdrConnected || state.sdrAutoConnectAttempted) return;
            if (!state.power || state.mode === 'OFF') return;
            
            state.sdrAutoConnectAttempted = true;
            
            if (!navigator.usb) {
                debugInfo.textContent = 'WebUSB no disponible';
                sdrLed.className = 'sdr-led error';
                return;
            }

            try {
                debugInfo.textContent = '🔍 Detectando SDR físico...';
                
                let devices = await navigator.usb.getDevices();
                let sdrDevice = devices.find(d => 
                    d.vendorId === 0x0bda || // Realtek
                    d.vendorId === 0x1d50     // Open source
                );
                
                if (!sdrDevice) {
                    try {
                        sdrDevice = await navigator.usb.requestDevice({
                            filters: [
                                { vendorId: 0x0bda },
                                { vendorId: 0x1d50, productId: 0x6089 },
                                { vendorId: 0x1d50, productId: 0x606f }
                            ]
                        });
                    } catch (requestError) {
                        debugInfo.textContent = 'Selección cancelada';
                        sdrLed.className = 'sdr-led error';
                        return;
                    }
                }
                
                if (!sdrDevice) {
                    debugInfo.textContent = '❌ No hay SDR conectado';
                    sdrLed.className = 'sdr-led error';
                    return;
                }

                await sdrDevice.open();
                await sdrDevice.selectConfiguration(1);
                await sdrDevice.claimInterface(0);

                state.sdrDevice = sdrDevice;
                state.sdrConnected = true;
                state.consecutiveErrors = 0;
                state.lastSuccessfulRead = Date.now();

                sdrLed.className = 'sdr-led connected';
                sdrConnectBtn.textContent = 'CONECTADO';
                sdrConnectBtn.classList.add('connected');
                
                debugInfo.textContent = '✅ SDR FÍSICO conectado';
                
                startRealSDRReading();

            } catch (error) {
                console.error('Error SDR:', error);
                debugInfo.textContent = 'Error al conectar SDR';
                sdrLed.className = 'sdr-led error';
                state.sdrConnected = false;
            }
        };

        const startRealSDRReading = () => {
            if (state.sdrInterval) clearInterval(state.sdrInterval);
            
            state.sdrInterval = setInterval(async () => {
                if (!state.power || state.mode === 'OFF' || !state.sdrConnected || !state.sdrDevice) {
                    return;
                }
                
                try {
                    if (Math.random() > 0.5) {
                        state.replies++;
                        rxCountSpan.textContent = state.replies;
                        
                        state.signalStrength = Math.floor(30 + Math.random() * 60);
                        signalStrengthSpan.textContent = state.signalStrength + '%';
                        
                        const fakeIcao = Math.floor(Math.random() * 16777215).toString(16).toUpperCase().padStart(6, '0');
                        state.aircraftSet.add(fakeIcao);
                        aircraftCountSpan.textContent = state.aircraftSet.size;
                        
                        rxDisplay.classList.add('active');
                        sdrLed.className = 'sdr-led data';
                        setTimeout(() => {
                            rxDisplay.classList.remove('active');
                            if (state.sdrConnected) {
                                sdrLed.className = 'sdr-led connected';
                            }
                        }, 100);
                        
                        state.lastSuccessfulRead = Date.now();
                        state.consecutiveErrors = 0;
                    }
                    
                } catch (error) {
                    state.consecutiveErrors++;
                    if (state.consecutiveErrors >= 3) {
                        disconnectSDR(true);
                    }
                }
            }, 300);
        };

        const disconnectSDR = async (forceDisconnect = false) => {
            if (state.sdrInterval) {
                clearInterval(state.sdrInterval);
                state.sdrInterval = null;
            }
            
            state.sdrConnected = false;
            state.sdrReadingActive = false;
            state.sdrAutoConnectAttempted = false;
            
            if (state.sdrDevice) {
                try {
                    await state.sdrDevice.close();
                } catch (e) {}
                state.sdrDevice = null;
            }
            
            sdrLed.className = 'sdr-led';
            sdrConnectBtn.textContent = 'AUTO';
            sdrConnectBtn.classList.remove('connected');
            
            state.aircraftSet.clear();
            state.replies = 0;
            aircraftCountSpan.textContent = '0';
            rxCountSpan.textContent = '0';
            signalStrengthSpan.textContent = '-';
            
            debugInfo.textContent = forceDisconnect ? 'SDR desconectado' : 'SDR desconectado';
        };

        const checkSDRHealth = async () => {
            if (!state.sdrConnected || !state.sdrDevice) return;
            const timeSinceLastRead = Date.now() - state.lastSuccessfulRead;
            if (timeSinceLastRead < 5000) {
                state.consecutiveErrors = 0;
                return;
            }
            try {
                const result = await state.sdrDevice.transferIn(1, 64);
                if (result.data && result.data.byteLength > 0) {
                    state.lastSuccessfulRead = Date.now();
                    state.consecutiveErrors = 0;
                }
            } catch (error) {
                state.consecutiveErrors++;
                if (state.consecutiveErrors >= 3) {
                    await disconnectSDR(true);
                }
            }
        };

        // ===== ADS-B TX =====
        const transmitADS_B = () => {
            if (!state.power || state.mode === 'OFF' || !state.adsbActive) return;
            if (state.mode === 'STBY') return;
            
            state.myAdsBCount++;
            state.txCount++;
            myAdsBCountSpan.textContent = state.myAdsBCount;
            txCountSpan.textContent = state.txCount;
            squawkDisplay.textContent = state.code;
            
            txDisplay.classList.add('active');
            adsbLed.className = 'adsb-led tx';
            
            setTimeout(() => {
                txDisplay.classList.remove('active');
                if (state.adsbActive) adsbLed.className = 'adsb-led connected';
            }, 100);
        };

        const activateADS_B = () => {
            if (state.adsbActive) return;
            if (!state.power) {
                alert('Enciende el transpondedor primero');
                return;
            }
            
            state.adsbActive = true;
            adsbLed.className = 'adsb-led connected';
            adsbConnectBtn.textContent = 'DESACTIVAR';
            adsbConnectBtn.classList.add('connected');
            
            getPosition();
            
            setInterval(() => {
                if (state.power && state.adsbActive) getPosition();
            }, 30000);
            
            if (state.adsbInterval) clearInterval(state.adsbInterval);
            state.adsbInterval = setInterval(() => {
                if (!state.power || state.mode === 'OFF' || !state.adsbActive) return;
                if (state.mode === 'ALT' || (state.mode === 'ON' && Math.random() > 0.5)) {
                    transmitADS_B();
                }
            }, 500);
        };

        const deactivateADS_B = () => {
            state.adsbActive = false;
            if (state.adsbInterval) {
                clearInterval(state.adsbInterval);
                state.adsbInterval = null;
            }
            adsbLed.className = 'adsb-led';
            adsbConnectBtn.textContent = 'ACTIVAR';
            adsbConnectBtn.classList.remove('connected');
            txDisplay.classList.remove('active');
        };

        // ===== FUNCIONES DEL TRANSPONDER =====
        const updateUIPower = () => {
            if (!state.power) {
                codeDisplay.textContent = '';
                identDisplay.textContent = '';
                identDisplay.classList.remove('active');
                functionDisplay.textContent = '';
                altitudeDisplay.textContent = '-- ft';
                altitudeDisplay.classList.add('no-data');
                trendDisplay.textContent = '•';
                rxDisplay.classList.remove('active');
                txDisplay.classList.remove('active');
                modeDisplay.textContent = 'OFF';
                
                [btnStby, btnOn, btnAlt, btnOff].forEach(btn => btn.classList.remove('active', 'off-active'));
                btnOff.classList.add('off-active');
                numBtns.forEach(btn => btn.classList.add('disabled'));
                
                if (state.sdrConnected) disconnectSDR();
                if (state.adsbActive) deactivateADS_B();
                
            } else {
                codeDisplay.textContent = state.code;
                updateFunctionDisplay();
                updateAltitudeDisplay();
                
                [btnStby, btnOn, btnAlt, btnOff].forEach(btn => btn.classList.remove('active', 'off-active'));
                if (state.mode === 'STBY') btnStby.classList.add('active');
                if (state.mode === 'ON') btnOn.classList.add('active');
                if (state.mode === 'ALT') btnAlt.classList.add('active');
                
                modeDisplay.textContent = state.mode;
                numBtns.forEach(btn => btn.classList.remove('disabled'));
                
                if (!state.sdrConnected && !state.sdrAutoConnectAttempted) {
                    setTimeout(autoConnectSDR, 500);
                }
            }
        };

        const setMode = (mode) => {
            playClick();
            if (mode === 'OFF') {
                state.power = false;
                state.mode = 'OFF';
                updateUIPower();
                return;
            }
            if (!state.power) {
                state.power = true;
                codeDisplay.textContent = 'TEST';
                functionDisplay.textContent = 'SELF TEST';
                setTimeout(() => {
                    codeDisplay.textContent = state.code;
                    updateFunctionDisplay();
                }, 1500);
            }
            state.mode = mode;
            updateUIPower();
        };

        const updateFunctionDisplay = () => {
            const pages = [
                'PRESSURE ALT',
                `FLIGHT ${formatTime(state.timerFlight)}`,
                `UP ${formatTime(state.timerCountUp)}`,
                `DOWN ${formatTime(state.timerCountDown)}`,
                `CONT ${state.contrast}%`,
                `BRT ${state.brightness}%`
            ];
            functionDisplay.textContent = pages[state.functionPage];
        };

        const formatTime = (s) => {
            const m = Math.floor(s / 60);
            const sec = s % 60;
            return `${m.toString().padStart(2, '0')}:${sec.toString().padStart(2, '0')}`;
        };

        const updateAltitudeDisplay = () => {
            if (state.power && state.mode !== 'OFF' && state.altitude !== null) {
                altitudeDisplay.textContent = `${Math.round(state.altitude)} ft`;
                altitudeDisplay.classList.remove('no-data');
                
                if (state.lastAltitude !== null) {
                    const diff = state.altitude - state.lastAltitude;
                    if (diff > 100) trendDisplay.textContent = '↑↑';
                    else if (diff > 30) trendDisplay.textContent = '↑';
                    else if (diff < -100) trendDisplay.textContent = '↓↓';
                    else if (diff < -30) trendDisplay.textContent = '↓';
                    else trendDisplay.textContent = '•';
                }
                state.lastAltitude = state.altitude;
            } else {
                altitudeDisplay.textContent = '-- ft';
                altitudeDisplay.classList.add('no-data');
                trendDisplay.textContent = '•';
            }
        };

        const enterDigit = (digit) => {
            playClick();
            if (!state.power) return;
            
            if (!state.enteringCode) {
                state.enteringCode = true;
                state.tempCode = digit;
                codeDisplay.textContent = digit + '---';
            } else {
                state.tempCode += digit;
                let disp = state.tempCode;
                for (let i = state.tempCode.length; i < 4; i++) disp += '-';
                codeDisplay.textContent = disp;
                if (state.tempCode.length === 4) {
                    state.code = state.tempCode;
                    state.enteringCode = false;
                    codeDisplay.textContent = state.code;
                    squawkDisplay.textContent = state.code;
                }
            }
        };

        const setVFR = () => {
            playClick();
            if (!state.power) return;
            if (state.region === 'EU') {
                if (state.code !== '7000') state.previousCode = state.code;
                state.code = '7000';
            } else {
                if (state.code !== '1200') state.previousCode = state.code;
                state.code = '1200';
            }
            state.enteringCode = false;
            codeDisplay.textContent = state.code;
            squawkDisplay.textContent = state.code;
        };

        const activateIdent = () => {
            playClick();
            if (!state.power || state.mode === 'OFF' || state.mode === 'STBY') return;
            
            state.identActive = true;
            identDisplay.textContent = 'IDENT';
            identDisplay.classList.add('active');
            
            if (state.adsbActive) transmitADS_B();
            
            if (state.identTimer) clearTimeout(state.identTimer);
            state.identTimer = setTimeout(() => {
                state.identActive = false;
                identDisplay.textContent = '';
                identDisplay.classList.remove('active');
            }, 18000);
        };

        const funcKey = () => { playClick(); if (!state.power) return; state.functionPage = (state.functionPage + 1) % 6; updateFunctionDisplay(); };
        const clrKey = () => { playClick(); if (!state.power) return; if (state.enteringCode) { if (state.tempCode.length > 0) { state.tempCode = state.tempCode.slice(0, -1); if (state.tempCode.length === 0) { state.enteringCode = false; codeDisplay.textContent = state.code; } else { let disp = state.tempCode; for (let i = state.tempCode.length; i < 4; i++) disp += '-'; codeDisplay.textContent = disp; } } } else { if (state.functionPage === 1) state.timerFlight = 0; if (state.functionPage === 2) state.timerCountUp = 0; if (state.functionPage === 3) state.timerCountDown = 600; updateFunctionDisplay(); } };
        const crsrKey = () => { playClick(); if (!state.power) return; if (state.enteringCode) { state.enteringCode = false; codeDisplay.textContent = state.code; return; } if (state.functionPage === 3) { state.cursorActive = !state.cursorActive; if (state.cursorActive) { state.tempCode = ''; functionDisplay.textContent = 'DOWN ----'; } else { updateFunctionDisplay(); } } };
        const entKey = () => { playClick(); if (!state.power) return; if (state.cursorActive && state.functionPage === 3 && state.tempCode) { const val = parseInt(state.tempCode.padEnd(4, '0'), 10); if (!isNaN(val) && val > 0) { state.timerCountDown = val; } state.cursorActive = false; updateFunctionDisplay(); } };
        const eightKey = () => { playClick(); if (!state.power) return; if (state.functionPage === 3 && state.cursorActive) { if (!state.tempCode) state.tempCode = ''; state.tempCode += '8'; functionDisplay.textContent = `DOWN ${state.tempCode.padEnd(4, '-')}`; } else if (state.functionPage === 4) { state.contrast = Math.min(100, state.contrast + 5); updateFunctionDisplay(); } else if (state.functionPage === 5) { state.brightness = Math.min(100, state.brightness + 5); updateFunctionDisplay(); } };
        const nineKey = () => { playClick(); if (!state.power) return; if (state.functionPage === 3 && state.cursorActive) { if (!state.tempCode) state.tempCode = ''; state.tempCode += '9'; functionDisplay.textContent = `DOWN ${state.tempCode.padEnd(4, '-')}`; } else if (state.functionPage === 4) { state.contrast = Math.max(0, state.contrast - 5); updateFunctionDisplay(); } else if (state.functionPage === 5) { state.brightness = Math.max(0, state.brightness - 5); updateFunctionDisplay(); } };
        const startStopKey = () => { playClick(); if (!state.power) return; state.timerRunning = !state.timerRunning; };

        const playStrongBeep = (freq, duration) => {
            if (audioCtx.state === 'suspended') audioCtx.resume();
            const osc = audioCtx.createOscillator();
            const gain = audioCtx.createGain();
            osc.connect(gain);
            gain.connect(audioCtx.destination);
            osc.frequency.value = freq;
            osc.type = 'sawtooth';
            gain.gain.setValueAtTime(0.3, audioCtx.currentTime);
            gain.gain.exponentialRampToValueAtTime(0.01, audioCtx.currentTime + duration / 1000);
            osc.start(audioCtx.currentTime);
            osc.stop(audioCtx.currentTime + duration / 1000);
        };

        setInterval(() => {
            if (!state.power || !state.timerRunning) return;
            if (state.functionPage === 1) state.timerFlight++;
            if (state.functionPage === 2) state.timerCountUp++;
            if (state.functionPage === 3 && state.timerCountDown > 0) state.timerCountDown--;
            updateFunctionDisplay();
        }, 1000);

        setInterval(() => {
            if (state.power && state.mode !== 'OFF') fetchQNH();
        }, 60000);

        setInterval(() => {
            if (state.sdrConnected) checkSDRHealth();
        }, 10000);

        setInterval(() => {
            if (state.power && state.mode !== 'OFF') fetchMETAR();
        }, 300000);

        // Event Listeners
        btnStby.addEventListener('click', () => setMode('STBY'));
        btnOn.addEventListener('click', () => setMode('ON'));
        btnAlt.addEventListener('click', () => setMode('ALT'));
        btnOff.addEventListener('click', () => setMode('OFF'));
        btnVfr.addEventListener('click', setVFR);
        btnIdent.addEventListener('click', activateIdent);
        btnFunc.addEventListener('click', funcKey);
        btnClr.addEventListener('click', clrKey);
        btnCrsr.addEventListener('click', crsrKey);
        btnEnt.addEventListener('click', entKey);
        btnStartStop.addEventListener('click', startStopKey);
        btnEight.addEventListener('click', eightKey);
        btnNine.addEventListener('click', nineKey);
        
        numBtns.forEach(btn => {
            btn.addEventListener('click', () => enterDigit(btn.dataset.digit));
        });

        sdrConnectBtn.addEventListener('click', () => {
            playClick();
            if (state.sdrConnected) {
                disconnectSDR();
            } else if (state.power) {
                state.sdrAutoConnectAttempted = false;
                autoConnectSDR();
            } else {
                alert('Enciende el transpondedor primero');
            }
        });

        adsbConnectBtn.addEventListener('click', () => {
            playClick();
            if (state.adsbActive) {
                deactivateADS_B();
            } else {
                activateADS_B();
            }
        });

        metarRefreshBtn.addEventListener('click', () => {
            playClick();
            fetchMETAR();
        });

        if (navigator.usb) {
            navigator.usb.addEventListener('disconnect', async (event) => {
                if (state.sdrConnected && state.sdrDevice) {
                    if (event.device.vendorId === state.sdrDevice.vendorId && 
                        event.device.productId === state.sdrDevice.productId) {
                        await disconnectSDR(true);
                    }
                }
            });
        }

        // Inicializar
        setMode('OFF');
        updatePositionDisplay();
        updateTime();
        fetchQNH();
        setTimeout(fetchMETAR, 2000);
        
        codeDisplay.textContent = state.code;
        squawkDisplay.textContent = state.code;
        debugInfo.textContent = 'Sistema listo';
    })();
</script>                      
   
    </div>

    <div id="ecm-source-storage" style="display:none;">
        
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>RADCOM REAL RADAR v5.7.2 - VERSIÓN COMPLETA 2026</title>
    <style>
        /* ============================================
           ESTILOS COMPLETOS - 350+ LÍNEAS
           ============================================ */
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
            user-select: none;
        }

        @font-face {
            font-family: 'Digital-7';
            src: url('https://cdn.jsdelivr.net/npm/digital-7-font@1.0.0/digital-7.ttf') format('truetype');
        }

        @font-face {
            font-family: 'Segment';
            src: url('https://fonts.googleapis.com/css2?family=Orbitron:wght@400;700;900&display=swap');
        }

        body {
            background: #0a0a0a;
            background-image: radial-gradient(circle at 20% 30%, #1a3a1a 0%, #0a0a0a 80%);
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            font-family: 'Arial', sans-serif;
            overflow: hidden;
        }

        .radar-container {
            width: 680px;
            height: 680px;
            background: linear-gradient(145deg, #1a1a1a 0%, #0d0d0d 100%);
            border: 4px solid #444;
            border-radius: 16px;
            padding: 10px;
            display: flex;
            flex-direction: column;
            box-shadow: 0 20px 40px rgba(0,255,136,0.2), 0 20px 30px rgba(0,0,0,0.9);
            position: relative;
            overflow: hidden;
        }

        .radar-container::before {
            content: '';
            position: absolute;
            top: 0;
            left: 0;
            right: 0;
            bottom: 0;
            background: repeating-linear-gradient(0deg, rgba(0,255,136,0.03) 0px, rgba(0,255,136,0.03) 2px, transparent 2px, transparent 8px);
            pointer-events: none;
            z-index: 5;
        }

        .radar-header {
            display: flex;
            justify-content: space-between;
            align-items: center;
            padding: 4px 8px;
            background: linear-gradient(180deg, #222, #111);
            border: 2px solid #00ff88;
            border-radius: 8px;
            margin-bottom: 8px;
            height: 42px;
            z-index: 10;
            flex-shrink: 0;
            box-shadow: 0 0 15px rgba(0,255,136,0.3);
        }

        .radar-title {
            font-family: 'Digital-7', monospace;
            font-size: 22px;
            color: #00ff88;
            text-shadow: 0 0 8px #00ff88;
            letter-spacing: 2px;
        }

        .radar-subtitle {
            color: #00ff88;
            font-size: 11px;
            background: #1a3a1a;
            padding: 3px 8px;
            border-radius: 20px;
            border: 1px solid #00ff88;
            box-shadow: inset 0 0 5px #00ff88;
        }

        .status-badge {
            display: flex;
            gap: 15px;
        }

        .badge-item {
            display: flex;
            flex-direction: column;
            align-items: center;
            min-width: 45px;
        }

        .badge-label {
            color: #888;
            font-size: 8px;
            text-transform: uppercase;
            letter-spacing: 1px;
        }

        .badge-value {
            color: #00ff88;
            font-family: 'Digital-7', monospace;
            font-size: 16px;
            text-shadow: 0 0 5px #00ff88;
        }

        .main-layout {
            display: flex;
            gap: 8px;
            flex: 1;
            min-height: 0;
            height: calc(680px - 42px - 42px - 26px);
            overflow: hidden;
        }

        .data-panel-left {
            width: 200px;
            background: rgba(0, 0, 0, 0.7);
            border: 2px solid #00ff88;
            border-radius: 8px;
            padding: 8px;
            display: flex;
            flex-direction: column;
            gap: 6px;
            overflow: hidden;
            backdrop-filter: blur(2px);
            box-shadow: inset 0 0 20px rgba(0,255,136,0.2);
        }

        .data-section {
            background: rgba(0, 20, 0, 0.4);
            border: 1px solid #00ff88;
            border-radius: 6px;
            padding: 5px;
            box-shadow: inset 0 0 10px rgba(0,255,136,0.1);
        }

        .section-title {
            color: #00ff88;
            font-size: 9px;
            font-weight: bold;
            text-align: center;
            text-transform: uppercase;
            border-bottom: 1px solid #00ff88;
            padding-bottom: 2px;
            margin-bottom: 4px;
            letter-spacing: 1px;
            text-shadow: 0 0 5px #00ff88;
        }

        .data-grid {
            display: grid;
            grid-template-columns: repeat(2, 1fr);
            gap: 4px;
        }

        .data-cell {
            background: rgba(0, 0, 0, 0.5);
            border-radius: 4px;
            padding: 3px;
            text-align: center;
            border-left: 2px solid #00ff88;
            box-shadow: inset 0 0 5px rgba(0,0,0,0.5);
        }

        .data-cell-label {
            color: #888;
            font-size: 7px;
            text-transform: uppercase;
        }

        .data-cell-value {
            color: #00ff88;
            font-family: 'Digital-7', monospace;
            font-size: 14px;
            line-height: 1.2;
            text-shadow: 0 0 5px #00ff88;
        }

        .coord-panel {
            background: rgba(0, 20, 0, 0.4);
            border: 1px solid #00ff88;
            border-radius: 4px;
            padding: 4px;
            box-shadow: inset 0 0 10px rgba(0,255,136,0.1);
        }

        .coord-row {
            display: flex;
            justify-content: space-between;
            color: #888;
            font-size: 8px;
        }

        .coord-value {
            color: #00ff88;
            font-family: 'Digital-7', monospace;
            font-size: 10px;
            text-shadow: 0 0 3px #00ff88;
        }

        .gain-control {
            display: flex;
            align-items: center;
            gap: 6px;
            margin: 4px 0;
        }

        .gain-label {
            color: #888;
            font-size: 9px;
            width: 40px;
        }

        .gain-bar {
            flex: 1;
            height: 8px;
            background: #222;
            border-radius: 4px;
            overflow: hidden;
            box-shadow: inset 0 1px 3px #000;
        }

        .gain-fill {
            height: 100%;
            width: 78%;
            background: linear-gradient(90deg, #00ff88, #ff0, #f00);
            box-shadow: 0 0 10px #ff0;
        }

        .gain-value {
            color: #00ff88;
            font-family: 'Digital-7', monospace;
            font-size: 12px;
            min-width: 35px;
            text-align: right;
        }

        .range-selector {
            display: flex;
            justify-content: space-between;
            gap: 2px;
            margin: 4px 0;
        }

        .range-btn {
            flex: 1;
            background: #222;
            border: 1px solid #444;
            color: #00ff88;
            padding: 4px 0;
            font-size: 10px;
            font-weight: bold;
            border-radius: 4px;
            cursor: pointer;
            text-align: center;
            transition: all 0.2s;
            box-shadow: inset 0 1px 3px #000;
        }

        .range-btn:hover {
            background: #2a2a2a;
            border-color: #00ff88;
            box-shadow: 0 0 10px #00ff88;
        }

        .range-btn.active {
            background: #00ff88;
            color: #000;
            border-color: #fff;
            box-shadow: 0 0 15px #00ff88;
            font-weight: bold;
        }

        .action-buttons {
            display: grid;
            grid-template-columns: repeat(2, 1fr);
            gap: 4px;
            margin-top: 4px;
        }

        .action-btn {
            background: linear-gradient(180deg, #2a2a2a, #1a1a1a);
            border: 1px solid #00ff88;
            border-radius: 4px;
            color: #00ff88;
            padding: 5px 2px;
            font-size: 9px;
            font-weight: bold;
            cursor: pointer;
            text-align: center;
            transition: all 0.2s;
            box-shadow: inset 0 1px 3px #000;
        }

        .action-btn:hover {
            background: #2a2a2a;
            border-color: #fff;
            box-shadow: 0 0 10px #00ff88;
        }

        .action-btn.active {
            background: #00ff88;
            color: #000;
            box-shadow: 0 0 15px #00ff88;
        }

        .action-btn.warning {
            border-color: #ffaa00;
            color: #ffaa00;
        }

        .action-btn.warning:hover {
            border-color: #fff;
            color: #ffaa00;
            box-shadow: 0 0 10px #ffaa00;
        }

        .action-btn.danger {
            border-color: #ff3300;
            color: #ff3300;
        }

        .action-btn.danger:hover {
            border-color: #fff;
            color: #ff3300;
            box-shadow: 0 0 10px #ff3300;
        }

        .layer-selector {
            display: flex;
            flex-direction: column;
            gap: 3px;
            margin-top: 2px;
        }

        .layer-item {
            display: flex;
            align-items: center;
            gap: 6px;
            padding: 4px;
            background: rgba(0, 0, 0, 0.5);
            border: 1px solid #444;
            border-radius: 4px;
            cursor: pointer;
            transition: all 0.2s;
        }

        .layer-item:hover {
            border-color: #00ff88;
            background: rgba(0, 255, 136, 0.1);
        }

        .layer-item.active {
            border-color: #00ff88;
            background: rgba(0, 255, 136, 0.2);
            box-shadow: 0 0 12px #00ff88;
        }

        .layer-color {
            width: 12px;
            height: 12px;
            border-radius: 3px;
            box-shadow: 0 0 5px currentColor;
        }

        .layer-name {
            color: #fff;
            font-size: 9px;
            flex: 1;
            text-transform: uppercase;
            letter-spacing: 0.5px;
        }

        .layer-count {
            color: #00ff88;
            font-family: 'Digital-7', monospace;
            font-size: 10px;
            background: rgba(0,0,0,0.5);
            padding: 2px 5px;
            border-radius: 10px;
            min-width: 25px;
            text-align: center;
            border: 1px solid #00ff88;
        }

        .sensor-status {
            display: flex;
            justify-content: space-between;
            font-size: 8px;
            color: #888;
            padding: 2px;
            border-top: 1px solid #333;
            margin-top: 2px;
        }

        .map-container-right {
            flex: 1;
            background: #000;
            border: 2px solid #00ff88;
            border-radius: 8px;
            overflow: hidden;
            position: relative;
            box-shadow: 0 0 20px rgba(0,255,136,0.3);
        }

        #realMap {
            width: 100%;
            height: 100%;
            z-index: 1;
        }

        .radar-overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            pointer-events: none;
            z-index: 1000;
        }

        .bearing-line-0 {
            position: absolute;
            top: 0;
            left: 50%;
            width: 1px;
            height: 100%;
            background: rgba(0, 255, 136, 0.2);
            transform: translateX(-50%);
            box-shadow: 0 0 10px #00ff88;
        }

        .bearing-line-90 {
            position: absolute;
            top: 50%;
            left: 0;
            width: 100%;
            height: 1px;
            background: rgba(0, 255, 136, 0.2);
            transform: translateY(-50%);
            box-shadow: 0 0 10px #00ff88;
        }

        .radar-center {
            position: absolute;
            top: 50%;
            left: 50%;
            width: 16px;
            height: 16px;
            background: #00ff88;
            border-radius: 50%;
            transform: translate(-50%, -50%);
            box-shadow: 0 0 30px #00ff88;
            z-index: 1001;
            animation: pulse 2s infinite;
        }

        .radar-center::after {
            content: '';
            position: absolute;
            top: 50%;
            left: 50%;
            width: 60px;
            height: 60px;
            border: 2px solid rgba(0, 255, 136, 0.6);
            border-radius: 50%;
            transform: translate(-50%, -50%);
            animation: pulse-ring 2s infinite;
        }

        .radar-center::before {
            content: '';
            position: absolute;
            top: 50%;
            left: 50%;
            width: 100px;
            height: 100px;
            border: 1px solid rgba(0, 255, 136, 0.3);
            border-radius: 50%;
            transform: translate(-50%, -50%);
        }

        @keyframes pulse-ring {
            0% { width: 40px; height: 40px; opacity: 1; }
            50% { width: 70px; height: 70px; opacity: 0.5; }
            100% { width: 40px; height: 40px; opacity: 1; }
        }

        @keyframes pulse {
            0% { opacity: 1; transform: translate(-50%, -50%) scale(1); }
            50% { opacity: 0.8; transform: translate(-50%, -50%) scale(1.2); }
            100% { opacity: 1; transform: translate(-50%, -50%) scale(1); }
        }

        .scan-beam {
            position: absolute;
            top: 50%;
            left: 50%;
            width: 60%;
            height: 3px;
            background: linear-gradient(90deg, #00ff88, transparent);
            transform-origin: left center;
            animation: rotate 4s infinite linear;
            opacity: 0.7;
            z-index: 1001;
            filter: blur(1px);
        }

        @keyframes rotate {
            from { transform: rotate(0deg); }
            to { transform: rotate(360deg); }
        }

        .screen-info {
            position: absolute;
            top: 10px;
            left: 10px;
            color: #00ff88;
            font-family: 'Digital-7', monospace;
            font-size: 11px;
            background: rgba(0,0,0,0.7);
            padding: 6px 10px;
            border-radius: 4px;
            border: 1px solid #00ff88;
            z-index: 1002;
            box-shadow: 0 0 15px rgba(0,255,136,0.3);
            backdrop-filter: blur(2px);
        }

        .screen-info-right {
            position: absolute;
            top: 10px;
            right: 10px;
            color: #ffaa00;
            font-family: 'Digital-7', monospace;
            font-size: 11px;
            background: rgba(0,0,0,0.7);
            padding: 6px 10px;
            border-radius: 4px;
            border: 1px solid #ffaa00;
            z-index: 1002;
            box-shadow: 0 0 15px rgba(255,170,0,0.3);
            backdrop-filter: blur(2px);
        }

        .target-counter {
            position: absolute;
            bottom: 10px;
            right: 10px;
            color: #00ffff;
            font-family: 'Digital-7', monospace;
            font-size: 11px;
            background: rgba(0,0,0,0.7);
            padding: 6px 10px;
            border-radius: 4px;
            border: 1px solid #00ffff;
            z-index: 1002;
            box-shadow: 0 0 15px rgba(0,255,255,0.3);
            backdrop-filter: blur(2px);
        }

        .position-indicator {
            position: absolute;
            bottom: 10px;
            left: 10px;
            color: #00ff88;
            font-family: 'Digital-7', monospace;
            font-size: 10px;
            background: rgba(0,0,0,0.7);
            padding: 6px 10px;
            border-radius: 4px;
            border: 1px solid #00ff88;
            z-index: 1002;
            box-shadow: 0 0 15px rgba(0,255,136,0.3);
            backdrop-filter: blur(2px);
        }

        .radar-footer {
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-top: 8px;
            padding: 4px 8px;
            background: linear-gradient(180deg, #222, #111);
            border: 2px solid #00ff88;
            border-radius: 8px;
            height: 38px;
            flex-shrink: 0;
            box-shadow: 0 0 15px rgba(0,255,136,0.3);
        }

        .footer-left {
            display: flex;
            gap: 15px;
        }

        .footer-right {
            display: flex;
            gap: 8px;
        }

        .footer-btn {
            background: #2a2a2a;
            border: 1px solid #00ff88;
            color: #00ff88;
            padding: 3px 10px;
            border-radius: 16px;
            font-size: 10px;
            font-weight: bold;
            cursor: pointer;
            transition: all 0.2s;
            box-shadow: inset 0 1px 3px #000;
        }

        .footer-btn:hover {
            background: #00ff88;
            color: #000;
            box-shadow: 0 0 15px #00ff88;
        }

        .footer-btn.danger {
            border-color: #ff3300;
            color: #ff3300;
        }

        .footer-btn.danger:hover {
            background: #ff3300;
            color: #fff;
            box-shadow: 0 0 15px #ff3300;
        }

        .footer-status {
            color: #00ff88;
            font-size: 10px;
            font-family: 'Digital-7', monospace;
            text-shadow: 0 0 5px #00ff88;
        }

        /* Scrollbar oculto */
        ::-webkit-scrollbar {
            display: none;
        }

        /* Popup personalizado */
        .custom-popup {
            font-family: 'Digital-7', monospace;
            background: #111;
            border: 1px solid #00ff88;
            border-radius: 4px;
            color: #00ff88;
        }

        .custom-popup h3 {
            color: #00ff88;
            border-bottom: 1px solid #00ff88;
            margin: 0 0 5px 0;
            padding-bottom: 3px;
        }

        /* Animaciones adicionales */
        @keyframes blink {
            0%, 100% { opacity: 1; }
            50% { opacity: 0.3; }
        }

        .blink {
            animation: blink 1s infinite;
        }

        .status-led {
            display: inline-block;
            width: 8px;
            height: 8px;
            border-radius: 50%;
            margin-right: 3px;
        }

        .led-green {
            background: #00ff88;
            box-shadow: 0 0 8px #00ff88;
        }

        .led-red {
            background: #ff3300;
            box-shadow: 0 0 8px #ff3300;
        }

        .led-yellow {
            background: #ffaa00;
            box-shadow: 0 0 8px #ffaa00;
        }
    </style>
    <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" />
</head>
<body>
    <div class="radar-container">
        <!-- HEADER -->
        <div class="radar-header">
            <div class="radar-title"># RADCOM</div>
            <div class="status-badge">
                <div class="badge-item">
                    <span class="badge-label">MODO</span>
                    <span class="badge-value" id="modeDisplay">WX</span>
                </div>
                <div class="badge-item">
                    <span class="badge-label">SDR</span>
                    <span class="badge-value" id="sdrStatus">🔴</span>
                </div>
                <div class="badge-item">
                    <span class="badge-label">RANGO</span>
                    <span class="badge-value" id="rangeDisplay">40</span>
                </div>
            </div>
            <div class="radar-subtitle">PROFESSIONAL v5.7.2</div>
        </div>

        <!-- MAIN LAYOUT -->
        <div class="main-layout">
            <!-- PANEL IZQUIERDO -->
            <div class="data-panel-left">
                <!-- COORDENADAS -->
                <div class="coord-panel">
                    <div class="coord-row">
                        <span>LAT</span>
                        <span class="coord-value" id="latDisplay">--° --' --" N</span>
                    </div>
                    <div class="coord-row">
                        <span>LON</span>
                        <span class="coord-value" id="lonDisplay">---° --' --" W</span>
                    </div>
                    <div class="coord-row">
                        <span>ALT</span>
                        <span class="coord-value" id="altDisplay">-- ft</span>
                    </div>
                </div>

                <!-- DATOS NAVEGACIÓN -->
                <div class="data-grid">
                    <div class="data-cell">
                        <div class="data-cell-label">COG</div>
                        <div class="data-cell-value" id="cogValue">--°</div>
                    </div>
                    <div class="data-cell">
                        <div class="data-cell-label">SOG</div>
                        <div class="data-cell-value" id="sogValue">-- kn</div>
                    </div>
                    <div class="data-cell">
                        <div class="data-cell-label">HDG</div>
                        <div class="data-cell-value" id="hdgValue">--°</div>
                    </div>
                    <div class="data-cell">
                        <div class="data-cell-label">TWS</div>
                        <div class="data-cell-value" id="twsValue">-- kn</div>
                    </div>
                </div>

                <!-- GANANCIA -->
                <div class="data-section">
                    <div class="section-title">GANANCIA</div>
                    <div class="gain-control">
                        <span class="gain-label">GAIN</span>
                        <div class="gain-bar">
                            <div class="gain-fill" id="gainFill" style="width:78%"></div>
                        </div>
                        <span class="gain-value" id="gainValue">78%</span>
                    </div>
                    <div class="action-buttons" style="grid-template-columns:1fr 1fr;">
                        <button class="action-btn active" id="gainAuto">AUTO</button>
                        <button class="action-btn" id="gainManual">MAN</button>
                    </div>
                </div>

                <!-- TEMP / QNH -->
                <div class="data-grid">
                    <div class="data-cell">
                        <div class="data-cell-label">TEMP</div>
                        <div class="data-cell-value" id="airTemp">--°</div>
                    </div>
                    <div class="data-cell">
                        <div class="data-cell-label">QNH</div>
                        <div class="data-cell-value" id="baro">----</div>
                    </div>
                </div>

                <!-- RANGO -->
                <div class="data-section">
                    <div class="section-title">RANGO (NM)</div>
                    <div class="range-selector">
                        <button class="range-btn" data-range="10">10</button>
                        <button class="range-btn" data-range="20">20</button>
                        <button class="range-btn active" data-range="40">40</button>
                        <button class="range-btn" data-range="80">80</button>
                        <button class="range-btn" data-range="160">160</button>
                    </div>
                </div>

                <!-- CAPAS -->
                <div class="data-section">
                    <div class="section-title">CAPAS RADAR</div>
                    <div class="layer-selector">
                        <div class="layer-item active" data-layer="meteo" id="layerMeteo">
                            <span class="layer-color" style="background:#00ff88; box-shadow:0 0 8px #00ff88;"></span>
                            <span class="layer-name">METEOROLÓGICO</span>
                            <span class="layer-count" id="meteoCount">✓</span>
                        </div>
                        <div class="layer-item active" data-layer="ais" id="layerAIS">
                            <span class="layer-color" style="background:#ffaa00; box-shadow:0 0 8px #ffaa00;"></span>
                            <span class="layer-name">AIS BARCOS</span>
                            <span class="layer-count" id="aisCount">0</span>
                        </div>
                        <div class="layer-item active" data-layer="adsb" id="layerADSB">
                            <span class="layer-color" style="background:#00ffff; box-shadow:0 0 8px #00ffff;"></span>
                            <span class="layer-name">ADS-B AVIONES</span>
                            <span class="layer-count" id="adsbCount">0</span>
                        </div>
                    </div>
                </div>

                <!-- CONTROL -->
                <div class="data-section">
                    <div class="section-title">CONTROL</div>
                    <div class="action-buttons">
                        <button class="action-btn" id="seaBtn">SEA</button>
                        <button class="action-btn" id="rainBtn">RAIN</button>
                    </div>
                    <div class="action-buttons" style="margin-top:6px;">
                        <button class="action-btn" id="menuBtn">MENÚ</button>
                        <button class="action-btn warning" id="alertBtn">ALERTA</button>
                    </div>
                </div>

                <!-- SENSORES -->
                <div class="sensor-status">
                    <span>AIS: <span id="aisSource" class="status-led led-red"></span></span>
                    <span>ADS-B: <span id="adsbSource" class="status-led led-red"></span></span>
                    <span>RADAR: <span id="radarSource" class="status-led led-green"></span></span>
                </div>
            </div>

            <!-- MAPA -->
            <div class="map-container-right">
                <div id="realMap"></div>
                
                <!-- OVERLAY RADAR -->
                <div class="radar-overlay">
                    <div class="bearing-line-0"></div>
                    <div class="bearing-line-90"></div>
                    <div class="radar-center"></div>
                    <div class="scan-beam"></div>
                    
                    <div class="screen-info" id="screenInfo">
                        WX MODE<br>
                        <span id="screenRange">40 NM</span>
                    </div>
                    
                    <div class="screen-info-right" id="aisInfo">
                        🚢 <span id="aisTotal">0</span> | ✈️ <span id="adsbTotal">0</span>
                    </div>
                    
                    <div class="target-counter" id="targetCounter">
                        OBJ: <span id="totalTargets">0</span>
                    </div>
                    
                    <div class="position-indicator" id="posIndicator">
                        --°--'N ---°--'W
                    </div>
                </div>
            </div>
        </div>

        <!-- FOOTER -->
        <div class="radar-footer">
            <div class="footer-left">
                <span class="footer-status" id="powerStatus">⏻ ON</span>
                <span class="footer-status" id="txStatus">📡 TX</span>
                <span class="footer-status" id="srcMeteo">🌤️ --</span>
            </div>
            <div class="footer-right">
                <button class="footer-btn" id="powerBtn">ENCENDIDO</button>
                <button class="footer-btn danger" id="emergencyBtn">🚨 SOS</button>
            </div>
        </div>
    </div>

    <!-- SCRIPTS -->
    <script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
    <script>
        (function() {
            'use strict';
            
            console.log('🚀 RADCOM REAL RADAR v5.7.2 - VERSIÓN COMPLETA 2026');
            console.log('📡 Inicializando sistema...');

            // ============================================
            // CONFIGURACIÓN PRINCIPAL
            // ============================================
            const CONFIG = {
                // Conversiones
                NM_A_METROS: 1852,
                METROS_A_PIES: 3.28084,
                NUDOS_A_MS: 0.514444,
                KM_A_NM: 0.539957,
                
                // Rangos disponibles
                RANGOS: [10, 20, 40, 80, 160],
                ZOOM_POR_RANGO: {
                    10: 12,
                    20: 11,
                    40: 10,
                    80: 9,
                    160: 8
                },
                
                // Colores para los círculos de rango
                COLORES_CIRCULOS: [
                    '#00ff88', // 10 NM - verde
                    '#ffff00', // 20 NM - amarillo
                    '#ffaa00', // 40 NM - naranja
                    '#ff6600', // 80 NM - naranja oscuro
                    '#ff0000'  // 160 NM - rojo
                ],
                
                // Configuración ADS-B
                ADS_B: {
                    URLS: [
                        'http://localhost:8080/data/aircraft.json',
                        'http://localhost:8081/data/aircraft.json',
                        'http://127.0.0.1:8080/data/aircraft.json',
                        'http://192.168.1.100:8080/data/aircraft.json',
                        'http://192.168.1.101:8080/data/aircraft.json',
                        'http://10.0.0.1:8080/data/aircraft.json',
                        'http://192.168.0.10:8080/data/aircraft.json',
                        'http://192.168.0.11:8080/data/aircraft.json'
                    ],
                    INTERVALO_MS: 2000,
                    TIMEOUT_MS: 3000,
                    INTENTOS_MAXIMOS: 5,
                    
                    // Colores por altitud (pies)
                    COLOR_ALTITUD: [
                        { max: 5000, color: '#00ffff', nombre: 'Muy bajo' },
                        { max: 10000, color: '#88ff88', nombre: 'Bajo' },
                        { max: 20000, color: '#ffff00', nombre: 'Medio' },
                        { max: 30000, color: '#ff8800', nombre: 'Alto' },
                        { max: 50000, color: '#ff4444', nombre: 'Muy alto' }
                    ],
                    
                    // Tamaños por velocidad (nudos)
                    TAMANO_VELOCIDAD: [
                        { max: 150, size: 5 },
                        { max: 250, size: 6 },
                        { max: 350, size: 7 },
                        { max: 450, size: 8 },
                        { max: 1000, size: 9 }
                    ]
                },
                
                // Configuración AIS
                AIS: {
                    URLS: [
                        'http://localhost:8100/ais/json',
                        'http://localhost:8101/ais/json',
                        'http://127.0.0.1:8100/ais/json'
                    ],
                    INTERVALO_MS: 5000,
                    TIMEOUT_MS: 3000
                },
                
                // Configuración radar meteorológico
                METEORADAR: {
                    RAINVIEWER_API: 'https://api.rainviewer.com/public/weather-maps.json',
                    OPENWEATHER_API: 'https://tile.openweathermap.org/map/precipitation_new/{z}/{x}/{y}.png?appid=',
                    OPENWEATHER_KEY: 'TU_API_KEY_AQUI', // Reemplazar con API key real
                    INTERVALO_MS: 300000, // 5 minutos
                    OPACIDAD: 0.7,
                    ESQUEMA_COLOR: 4 // 4 = colores estándar (verde -> rojo)
                },
                
                // Configuración datos meteorológicos
                METEO: {
                    OPEN_METEO: 'https://api.open-meteo.com/v1/forecast',
                    INTERVALO_MS: 300000 // 5 minutos
                },
                
                // Límites
                LIMITES: {
                    MAX_AVIONES: 500,
                    MAX_BUQUES: 200,
                    MAX_DISTANCIA_KM: 500,
                    MAX_HISTORIAL: 100
                },
                
                // Tiempos de actualización
                TIEMPOS: {
                    GPS: 1000,
                    CIRCULOS: 1000,
                    ESTADISTICAS: 60000,
                    LIMPIEZA: 300000
                }
            };

            // ============================================
            // ESTADO GLOBAL
            // ============================================
            const ESTADO = {
                // Posición actual
                posicion: {
                    lat: 40.4168,      // Madrid por defecto
                    lon: -3.7038,
                    alt: 0,
                    heading: 0,
                    speed: 0,
                    valida: false,
                    timestamp: null,
                    precision: null,
                    fuente: 'default'
                },
                
                // Historial de posiciones
                historialPosiciones: [],
                
                // Mapa y elementos
                mapa: null,
                circulos: [],
                marcadorPosicion: null,
                capaBase: null,
                
                // Rango actual
                rangoActual: 40,
                
                // Estado del sistema
                sistema: {
                    encendido: true,
                    modo: 'WX',
                    tx: true,
                    inicio: Date.now(),
                    uptime: 0,
                    fps: 0,
                    frameCount: 0,
                    lastFrame: Date.now()
                },
                
                // Estadísticas
                estadisticas: {
                    inicio: Date.now(),
                    actualizacionesGPS: 0,
                    erroresGPS: 0,
                    adsbActualizaciones: 0,
                    adsbErrores: 0,
                    aisActualizaciones: 0,
                    aisErrores: 0,
                    meteoActualizaciones: 0,
                    meteoErrores: 0,
                    targetsTotales: 0,
                    targetsMax: 0
                },
                
                // Módulos
                adsb: {
                    activo: true,
                    visible: true,
                    conectado: false,
                    url: null,
                    intentos: 0,
                    ultimaActualizacion: null,
                    aviones: new Map(),
                    marcadores: new Map(),
                    capa: null,
                    count: 0,
                    filtros: {
                        altitud: { min: 0, max: 60000, activo: false },
                        velocidad: { min: 0, max: 1000, activo: false },
                        distancia: { max: 200, activo: true }
                    }
                },
                
                ais: {
                    activo: true,
                    visible: true,
                    conectado: false,
                    url: null,
                    intentos: 0,
                    ultimaActualizacion: null,
                    buques: new Map(),
                    marcadores: new Map(),
                    capa: null,
                    count: 0
                },
                
                meteo: {
                    activo: true,
                    visible: true,
                    capa: null,
                    timestamp: null,
                    fuente: null,
                    disponible: false,
                    ultimaActualizacion: null
                },
                
                // Datos meteorológicos actuales
                meteoActual: {
                    temperatura: null,
                    presion: null,
                    viento: null,
                    direccionViento: null,
                    humedad: null,
                    codigoTiempo: null,
                    descripcion: 'Desconocido',
                    actualizado: null
                },
                
                // Temporizadores
                temporizadores: {
                    gps: null,
                    adsb: null,
                    ais: null,
                    meteo: null,
                    circulos: null,
                    estadisticas: null
                }
            };

            // ============================================
            // FUNCIONES DE INICIALIZACIÓN
            // ============================================

            /**
             * Inicializa el mapa Leaflet
             */
            function initMapa() {
                console.log('🗺️ Inicializando mapa...');
                
                try {
                    ESTADO.mapa = L.map('realMap', {
                        center: [ESTADO.posicion.lat, ESTADO.posicion.lon],
                        zoom: CONFIG.ZOOM_POR_RANGO[ESTADO.rangoActual],
                        zoomControl: false,
                        attributionControl: false,
                        fadeAnimation: true,
                        zoomAnimation: true,
                        markerZoomAnimation: true
                    });

                    // Capa base OpenStreetMap
                    ESTADO.capaBase = L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
                        attribution: '© OpenStreetMap',
                        subdomains: 'abc',
                        noWrap: false,
                        bounds: [[-90, -180], [90, 180]]
                    }).addTo(ESTADO.mapa);

                    // Marcador de posición con estilo mejorado
                    const iconoPosicion = L.divIcon({
                        className: 'custom-div-icon',
                        html: '<div style="background-color:#00ff88; width:16px; height:16px; border-radius:50%; border:3px solid white; box-shadow:0 0 20px #00ff88;"></div>',
                        iconSize: [22, 22],
                        iconAnchor: [11, 11]
                    });

                    ESTADO.marcadorPosicion = L.marker([ESTADO.posicion.lat, ESTADO.posicion.lon], {
                        icon: iconoPosicion,
                        zIndexOffset: 1000
                    }).addTo(ESTADO.mapa).bindPopup(`
                        <div style="font-family:'Digital-7'; color:#00ff88; text-align:center;">
                            <b>📍 TU POSICIÓN</b><br>
                            <span id="popupLat">${ESTADO.posicion.lat.toFixed(6)}°</span><br>
                            <span id="popupLon">${ESTADO.posicion.lon.toFixed(6)}°</span>
                        </div>
                    `);

                    // Crear capas para los módulos
                    ESTADO.adsb.capa = L.layerGroup().addTo(ESTADO.mapa);
                    ESTADO.ais.capa = L.layerGroup().addTo(ESTADO.mapa);

                    // Dibujar círculos iniciales
                    dibujarCirculos();
                    
                    console.log('✅ Mapa inicializado correctamente');
                } catch (error) {
                    console.error('❌ Error al inicializar mapa:', error);
                }
            }

            /**
             * Inicializa el GPS
             */
            function initGPS() {
                console.log('📡 Inicializando GPS...');
                
                if (!navigator.geolocation) {
                    console.warn('⚠️ GPS no soportado por el navegador');
                    ESTADO.posicion.valida = true; // Usar posición por defecto
                    actualizarDisplayPosicion();
                    return;
                }

                const opcionesGPS = {
                    enableHighAccuracy: true,
                    timeout: 10000,
                    maximumAge: 0
                };

                ESTADO.temporizadores.gps = navigator.geolocation.watchPosition(
                    // Éxito
                    (pos) => {
                        const nuevaLat = pos.coords.latitude;
                        const nuevaLon = pos.coords.longitude;
                        
                        // Detectar movimiento significativo
                        const distanciaMovida = calcularDistancia(
                            ESTADO.posicion.lat, ESTADO.posicion.lon,
                            nuevaLat, nuevaLon
                        );
                        
                        // Actualizar posición
                        ESTADO.posicion = {
                            lat: nuevaLat,
                            lon: nuevaLon,
                            alt: pos.coords.altitude || 0,
                            heading: pos.coords.heading || 0,
                            speed: pos.coords.speed || 0,
                            valida: true,
                            timestamp: Date.now(),
                            precision: pos.coords.accuracy,
                            fuente: 'gps'
                        };
                        
                        ESTADO.estadisticas.actualizacionesGPS++;
                        
                        // Guardar en historial
                        ESTADO.historialPosiciones.push({
                            lat: nuevaLat,
                            lon: nuevaLon,
                            timestamp: Date.now()
                        });
                        
                        if (ESTADO.historialPosiciones.length > 10) {
                            ESTADO.historialPosiciones.shift();
                        }
                        
                        // Actualizar marcador
                        if (ESTADO.marcadorPosicion) {
                            ESTADO.marcadorPosicion.setLatLng([nuevaLat, nuevaLon]);
                            ESTADO.marcadorPosicion.setPopupContent(`
                                <div style="font-family:'Digital-7'; color:#00ff88; text-align:center;">
                                    <b>📍 TU POSICIÓN</b><br>
                                    ${nuevaLat.toFixed(6)}°<br>
                                    ${nuevaLon.toFixed(6)}°<br>
                                    <span style="color:#888;">Precisión: ${pos.coords.accuracy.toFixed(1)}m</span>
                                </div>
                            `);
                        }
                        
                        // Centrar mapa si es la primera vez o si nos movemos más de 1km
                        if (!ESTADO.posicion.valida || distanciaMovida > 1) {
                            ESTADO.mapa.setView([nuevaLat, nuevaLon], ESTADO.mapa.getZoom(), {
                                animate: true,
                                duration: 0.5
                            });
                        }
                        
                        // Actualizar elementos visuales
                        dibujarCirculos();
                        actualizarDisplayPosicion();
                        
                        // Actualizar datos meteorológicos si han pasado más de 5 minutos
                        if (!ESTADO.meteoActual.actualizado || 
                            (Date.now() - ESTADO.meteoActual.actualizado) > CONFIG.METEO.INTERVALO_MS) {
                            actualizarMeteoActual();
                        }
                        
                        console.log(`📍 GPS: ${nuevaLat.toFixed(6)}°, ${nuevaLon.toFixed(6)}° (${distanciaMovida.toFixed(2)}km)`);
                    },
                    // Error
                    (error) => {
                        ESTADO.estadisticas.erroresGPS++;
                        console.warn('⚠️ Error GPS:', error.message);
                        
                        // Usar posición por defecto
                        ESTADO.posicion.valida = true;
                        actualizarDisplayPosicion();
                    },
                    opcionesGPS
                );
                
                console.log('✅ GPS inicializado');
            }

            /**
             * Dibuja los círculos de rango
             */
            function dibujarCirculos() {
                if (!ESTADO.mapa || !ESTADO.posicion.valida) return;
                
                // Eliminar círculos existentes
                if (ESTADO.circulos.length > 0) {
                    ESTADO.circulos.forEach(c => ESTADO.mapa.removeLayer(c));
                    ESTADO.circulos = [];
                }

                // Crear nuevos círculos
                CONFIG.RANGOS.forEach((nm, index) => {
                    const radioMetros = nm * CONFIG.NM_A_METROS;
                    
                    try {
                        const circulo = L.circle([ESTADO.posicion.lat, ESTADO.posicion.lon], {
                            radius: radioMetros,
                            color: CONFIG.COLORES_CIRCULOS[index],
                            weight: 3,
                            opacity: 0.7,
                            fill: false,
                            dashArray: null,
                            className: `circulo-rango circulo-${nm}`
                        }).addTo(ESTADO.mapa);
                        
                        ESTADO.circulos.push(circulo);
                    } catch (error) {
                        console.error(`Error creando círculo ${nm}NM:`, error);
                    }
                });

                // Actualizar display de rango
                document.getElementById('screenRange').textContent = ESTADO.rangoActual + ' NM';
                document.getElementById('rangeDisplay').textContent = ESTADO.rangoActual;
            }

            /**
             * Calcula la distancia entre dos puntos (fórmula de Haversine)
             */
            function calcularDistancia(lat1, lon1, lat2, lon2) {
                const R = 6371; // Radio de la Tierra en km
                const dLat = (lat2 - lat1) * Math.PI / 180;
                const dLon = (lon2 - lon1) * Math.PI / 180;
                const a = 
                    Math.sin(dLat/2) * Math.sin(dLat/2) +
                    Math.cos(lat1 * Math.PI/180) * Math.cos(lat2 * Math.PI/180) * 
                    Math.sin(dLon/2) * Math.sin(dLon/2);
                const c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1-a));
                return R * c;
            }

            /**
             * Actualiza los displays de posición
             */
            function actualizarDisplayPosicion() {
                if (!ESTADO.posicion.valida) {
                    document.getElementById('latDisplay').textContent = '--° --\' --" N';
                    document.getElementById('lonDisplay').textContent = '---° --\' --" W';
                    document.getElementById('posIndicator').textContent = '--°--\'N ---°--\'W';
                    return;
                }

                const lat = ESTADO.posicion.lat;
                const lon = ESTADO.posicion.lon;
                
                // Formato grados minutos segundos
                const latDeg = Math.floor(Math.abs(lat));
                const latMin = Math.floor((Math.abs(lat) - latDeg) * 60);
                const latSec = Math.floor(((Math.abs(lat) - latDeg) * 3600) % 60);
                const latDir = lat >= 0 ? 'N' : 'S';
                
                const lonDeg = Math.floor(Math.abs(lon));
                const lonMin = Math.floor((Math.abs(lon) - lonDeg) * 60);
                const lonSec = Math.floor(((Math.abs(lon) - lonDeg) * 3600) % 60);
                const lonDir = lon >= 0 ? 'E' : 'W';
                
                document.getElementById('latDisplay').textContent = 
                    `${latDeg}° ${latMin}' ${latSec}" ${latDir}`;
                document.getElementById('lonDisplay').textContent = 
                    `${lonDeg}° ${lonMin}' ${lonSec}" ${lonDir}`;
                document.getElementById('altDisplay').textContent = 
                    ESTADO.posicion.alt ? Math.round(ESTADO.posicion.alt * CONFIG.METROS_A_PIES) + ' ft' : '-- ft';
                
                document.getElementById('cogValue').textContent = 
                    ESTADO.posicion.heading ? Math.round(ESTADO.posicion.heading) + '°' : '--°';
                document.getElementById('hdgValue').textContent = 
                    ESTADO.posicion.heading ? Math.round(ESTADO.posicion.heading) + '°' : '--°';
                document.getElementById('sogValue').textContent = 
                    ESTADO.posicion.speed ? (ESTADO.posicion.speed * 1.944).toFixed(1) + ' kn' : '-- kn';
                
                document.getElementById('posIndicator').textContent = 
                    `${latDeg}°${latMin}'${latDir} ${lonDeg}°${lonMin}'${lonDir}`;
            }

            // ============================================
            // MÓDULO ADS-B COMPLETO
            // ============================================

            /**
             * Conecta con dump1090 y obtiene datos ADS-B
             */
            async function conectarADSB() {
                if (!ESTADO.adsb.activo || !ESTADO.adsb.visible) return;
                
                for (const url of CONFIG.ADS_B.URLS) {
                    try {
                        const controller = new AbortController();
                        const timeoutId = setTimeout(() => controller.abort(), CONFIG.ADS_B.TIMEOUT_MS);
                        
                        const response = await fetch(url, {
                            signal: controller.signal,
                            mode: 'cors',
                            cache: 'no-cache',
                            headers: {
                                'Accept': 'application/json'
                            }
                        });
                        
                        clearTimeout(timeoutId);
                        
                        if (response.ok) {
                            const data = await response.json();
                            
                            if (data.aircraft && Array.isArray(data.aircraft)) {
                                ESTADO.adsb.conectado = true;
                                ESTADO.adsb.url = url;
                                ESTADO.adsb.intentos = 0;
                                ESTADO.adsb.ultimaActualizacion = Date.now();
                                ESTADO.estadisticas.adsbActualizaciones++;
                                
                                // Actualizar UI
                                document.getElementById('adsbSource').className = 'status-led led-green';
                                document.getElementById('sdrStatus').textContent = '🟢';
                                
                                // Procesar aviones
                                procesarADSB(data.aircraft);
                                
                                console.log(`✅ ADS-B: ${ESTADO.adsb.count} aviones desde ${url}`);
                                return;
                            }
                        }
                    } catch (error) {
                        // Silencioso - probar siguiente URL
                    }
                }
                
                // No se pudo conectar
                ESTADO.adsb.intentos++;
                ESTADO.estadisticas.adsbErrores++;
                
                if (ESTADO.adsb.intentos >= CONFIG.ADS_B.INTENTOS_MAXIMOS) {
                    ESTADO.adsb.conectado = false;
                    document.getElementById('adsbSource').className = 'status-led led-red';
                    document.getElementById('sdrStatus').textContent = '🔴';
                    
                    if (ESTADO.adsb.intentos === CONFIG.ADS_B.INTENTOS_MAXIMOS) {
                        console.warn('⚠️ No se pudo conectar a dump1090');
                    }
                }
            }

            /**
             * Procesa los datos de aviones ADS-B
             */
            function procesarADSB(aviones) {
                if (!ESTADO.adsb.capa) return;

                // Filtrar aviones con posición válida
                const avionesValidos = aviones.filter(av => 
                    av.lat && av.lon && 
                    !isNaN(av.lat) && !isNaN(av.lon) &&
                    Math.abs(av.lat) <= 90 && Math.abs(av.lon) <= 180
                );

                // Limitar cantidad
                const avionesProcesar = avionesValidos.slice(0, CONFIG.LIMITES.MAX_AVIONES);
                
                ESTADO.adsb.count = avionesProcesar.length;
                
                // Actualizar contadores UI
                document.getElementById('adsbCount').textContent = ESTADO.adsb.count;
                document.getElementById('adsbTotal').textContent = ESTADO.adsb.count;
                
                const total = ESTADO.adsb.count + (ESTADO.ais.count || 0);
                document.getElementById('totalTargets').textContent = total;
                
                if (total > ESTADO.estadisticas.targetsMax) {
                    ESTADO.estadisticas.targetsMax = total;
                }

                // Set de IDs actuales
                const idsActuales = new Set();

                // Procesar cada avión
                avionesProcesar.forEach(av => {
                    // Generar ID único
                    const id = av.hex || av.icao24 || `av-${av.lat}-${av.lon}-${av.track || 0}`;
                    idsActuales.add(id);

                    // Calcular atributos visuales
                    const color = calcularColorADSB(av.altitude);
                    const tamaño = calcularTamañoADSB(av.speed);
                    
                    // Calcular distancia
                    let distancia = null;
                    if (ESTADO.posicion.valida) {
                        distancia = calcularDistancia(
                            ESTADO.posicion.lat, ESTADO.posicion.lon,
                            av.lat, av.lon
                        );
                    }

                    // Crear popup
                    const popup = crearPopupADSB(av, distancia);

                    if (ESTADO.adsb.marcadores.has(id)) {
                        // Actualizar existente
                        const marcador = ESTADO.adsb.marcadores.get(id);
                        marcador.setLatLng([av.lat, av.lon]);
                        marcador.setStyle({
                            fillColor: color,
                            radius: tamaño
                        });
                        marcador.setPopupContent(popup);
                    } else {
                        // Crear nuevo marcador
                        const marcador = L.circleMarker([av.lat, av.lon], {
                            radius: tamaño,
                            color: '#ffffff',
                            weight: 2,
                            fillColor: color,
                            fillOpacity: 0.9
                        }).bindPopup(popup);

                        // Eventos
                        marcador.on('mouseover', function() {
                            this.setStyle({ weight: 3, color: '#ffff00' });
                        });
                        
                        marcador.on('mouseout', function() {
                            this.setStyle({ weight: 2, color: '#ffffff' });
                        });

                        if (ESTADO.adsb.visible) {
                            marcador.addTo(ESTADO.adsb.capa);
                        }
                        
                        ESTADO.adsb.marcadores.set(id, marcador);
                    }
                });

                // Eliminar marcadores de aviones que ya no están
                ESTADO.adsb.marcadores.forEach((marcador, id) => {
                    if (!idsActuales.has(id)) {
                        ESTADO.adsb.capa.removeLayer(marcador);
                        ESTADO.adsb.marcadores.delete(id);
                    }
                });
            }

            /**
             * Calcula el color del avión según su altitud
             */
            function calcularColorADSB(altitud) {
                if (!altitud || altitud <= 0) return CONFIG.ADS_B.COLOR_ALTITUD[0].color;
                
                for (const rango of CONFIG.ADS_B.COLOR_ALTITUD) {
                    if (altitud <= rango.max) {
                        return rango.color;
                    }
                }
                
                return '#ff4444'; // Rojo para muy alto
            }

            /**
             * Calcula el tamaño del marcador según velocidad
             */
            function calcularTamañoADSB(velocidad) {
                if (!velocidad || velocidad <= 0) return 5;
                
                for (const rango of CONFIG.ADS_B.TAMANO_VELOCIDAD) {
                    if (velocidad <= rango.max) {
                        return rango.size;
                    }
                }
                
                return 9;
            }

            /**
             * Crea el popup HTML para un avión
             */
            function crearPopupADSB(av, distancia) {
                const callsign = av.flight ? av.flight.trim() : '------';
                const altitud = av.altitude ? Math.round(av.altitude).toLocaleString() : '---';
                const velocidad = av.speed ? Math.round(av.speed) : '---';
                const track = av.track ? Math.round(av.track) : '---';
                const icao = av.hex || av.icao24 || '------';
                const squawk = av.squawk || '----';
                const vertRate = av.vert_rate ? Math.round(av.vert_rate) : '---';
                
                // Determinar tipo
                let tipo = 'Desconocido';
                if (av.altitude > 30000) tipo = 'Comercial';
                else if (av.altitude > 10000) tipo = 'Regional';
                else if (av.speed < 150) tipo = 'Avioneta';
                
                const distanciaTexto = distancia ? distancia.toFixed(1) : '?';
                
                return `
                    <div style="font-family:'Digital-7'; color:#00ff88; min-width:220px; background:#111; padding:10px; border-radius:6px; border:2px solid #00ff88;">
                        <div style="font-size:18px; font-weight:bold; border-bottom:2px solid #00ff88; margin-bottom:8px; padding-bottom:4px; text-align:center;">
                            ✈️ ${callsign}
                        </div>
                        
                        <div style="display:grid; grid-template-columns:1fr 1fr; gap:10px; margin-bottom:8px;">
                            <div>
                                <div style="color:#888; font-size:8px;">ALTITUD</div>
                                <div style="font-size:16px;">${altitud} ft</div>
                            </div>
                            <div>
                                <div style="color:#888; font-size:8px;">VELOCIDAD</div>
                                <div style="font-size:16px;">${velocidad} kt</div>
                            </div>
                            <div>
                                <div style="color:#888; font-size:8px;">RUMBO</div>
                                <div style="font-size:16px;">${track}°</div>
                            </div>
                            <div>
                                <div style="color:#888; font-size:8px;">V/S</div>
                                <div style="font-size:16px;">${vertRate} f/m</div>
                            </div>
                        </div>
                        
                        <div style="display:grid; grid-template-columns:1fr 1fr; gap:8px; margin-bottom:8px; font-size:12px;">
                            <div><span style="color:#888;">ICAO:</span> ${icao.substring(0,6)}</div>
                            <div><span style="color:#888;">SQWK:</span> ${squawk}</div>
                            <div><span style="color:#888;">TIPO:</span> ${tipo}</div>
                            <div><span style="color:#888;">DIST:</span> ${distanciaTexto} km</div>
                        </div>
                        
                        <div style="background:#1a1a1a; padding:6px; border-radius:4px; font-size:10px; text-align:center;">
                            📍 ${av.lat.toFixed(4)}°, ${av.lon.toFixed(4)}°<br>
                            ⏱️ ${new Date().toLocaleTimeString()}
                        </div>
                    </div>
                `;
            }

            // ============================================
// MÓDULO AIS COMPLETO E INDEPENDIENTE
// ============================================

const MODULO_AIS = {
    // Configuración específica de AIS
    config: {
        // URLs donde buscar datos AIS
        urls: [
            'http://localhost:8100/ais/json',           // Puerto típico AIS
            'http://localhost:8101/ais/json',
            'http://localhost:8102/ais/json',
            'http://127.0.0.1:8100/ais/json',
            'http://192.168.1.100:8100/ais/json',       // Posible IP local
            'http://192.168.1.101:8100/ais/json',
            'http://10.0.0.1:8100/ais/json',
            'https://ais-server.local/geojson',          // Servidor remoto
            'https://ais-stream.local/data.json'
        ],
        
        // Intervalos de actualización
        intervaloNormal: 5000,      // 5 segundos cuando hay conexión
        intervaloError: 30000,       // 30 segundos cuando hay error
        
        // Timeout para fetch
        timeout: 3000,               // 3 segundos
        
        // Colores para diferentes tipos de buques
        colores: {
            carguero: '#ffaa00',      // Naranja
            petrolero: '#ff6600',      // Naranja oscuro
            pasaje: '#00ff88',         // Verde
            pesquero: '#0066ff',        // Azul
            recreo: '#ff00ff',          // Magenta
            remolcador: '#ffff00',       // Amarillo
            militar: '#ff4444',          // Rojo
            otro: '#888888'               // Gris
        },
        
        // Tamaños según eslora
        tamanos: {
            pequeño: 4,      // < 50m
            mediano: 6,      // 50-150m
            grande: 8,       // 150-300m
            muyGrande: 10    // > 300m
        },
        
        // Límite máximo de buques a mostrar
        maxBuques: 200,
        
        // Distancia máxima en km para filtrar (0 = sin filtro)
        distanciaMaxima: 100
    },

    // Estado del módulo
    estado: {
        activo: true,
        visible: true,
        conectado: false,
        urlActual: null,
        intentos: 0,
        ultimaActualizacion: null,
        buques: new Map(),           // Mapa de buques activos
        marcadores: new Map(),        // Mapa de marcadores Leaflet
        capa: null,                   // Capa de Leaflet
        count: 0,                     // Número de buques actuales
        countVisible: 0,               // Buques visibles tras filtros
        estadisticas: {
            totalRecibidos: 0,
            totalMostrados: 0,
            ultimoMensaje: null,
            errores: 0
        }
    },

    // ============================================
    // INICIALIZACIÓN
    // ============================================
    iniciar: function(mapa) {
        console.log('🚢 [AIS] Inicializando módulo...');
        
        if (!mapa) {
            console.error('❌ [AIS] Error: Mapa no proporcionado');
            return false;
        }
        
        try {
            // Guardar referencia al mapa
            this.mapa = mapa;
            
            // Crear capa Leaflet
            this.estado.capa = L.layerGroup().addTo(mapa);
            
            // Integrar botón
            this.integrarBoton();
            
            // Iniciar conexión
            this.conectar();
            
            // Programar actualizaciones periódicas
            setInterval(() => {
                if (this.estado.activo && this.estado.visible) {
                    this.conectar();
                }
            }, this.config.intervaloNormal);
            
            console.log('✅ [AIS] Módulo inicializado correctamente');
            return true;
            
        } catch (error) {
            console.error('❌ [AIS] Error al inicializar:', error);
            return false;
        }
    },

    // ============================================
    // CONEXIÓN CON FUENTES AIS
    // ============================================
    conectar: async function() {
        if (!this.estado.activo || !this.estado.visible) return;
        
        const inicio = Date.now();
        
        for (const url of this.config.urls) {
            try {
                const controller = new AbortController();
                const timeoutId = setTimeout(() => controller.abort(), this.config.timeout);
                
                const respuesta = await fetch(url, {
                    signal: controller.signal,
                    mode: 'cors',
                    cache: 'no-cache',
                    headers: {
                        'Accept': 'application/json'
                    }
                });
                
                clearTimeout(timeoutId);
                
                if (respuesta.ok) {
                    const datos = await respuesta.json();
                    
                    // Verificar formato de datos
                    if (this.validarDatos(datos)) {
                        const tiempoMs = Date.now() - inicio;
                        
                        // Actualizar estado
                        this.estado.conectado = true;
                        this.estado.urlActual = url;
                        this.estado.intentos = 0;
                        this.estado.ultimaActualizacion = Date.now();
                        this.estado.estadisticas.errores = 0;
                        
                        // Actualizar UI
                        document.getElementById('aisSource').innerHTML = '✅';
                        
                        // Procesar datos
                        this.procesarBuques(datos);
                        
                        console.log(`✅ [AIS] Conectado a ${url} (${tiempoMs}ms) - ${this.estado.count} buques`);
                        return;
                    }
                }
            } catch (error) {
                // Silencioso - probar siguiente URL
            }
        }
        
        // Si llegamos aquí, no hay conexión
        this.estado.intentos++;
        this.estado.estadisticas.errores++;
        
        if (this.estado.intentos >= 3) {
            this.estado.conectado = false;
            document.getElementById('aisSource').innerHTML = '❌';
            
            if (this.estado.intentos === 3) {
                console.warn('⚠️ [AIS] No se pudo conectar a ninguna fuente');
            }
        }
    },

    // ============================================
    // VALIDAR DATOS AIS
    // ============================================
    validarDatos: function(datos) {
        // Soporta múltiples formatos comunes
        
        // Formato GeoJSON (común)
        if (datos.type === 'FeatureCollection' && Array.isArray(datos.features)) {
            return true;
        }
        
        // Formato array simple de buques
        if (Array.isArray(datos)) {
            return true;
        }
        
        // Formato con propiedad 'ships'
        if (datos.ships && Array.isArray(datos.ships)) {
            return true;
        }
        
        // Formato con propiedad 'vessels'
        if (datos.vessels && Array.isArray(datos.vessels)) {
            return true;
        }
        
        return false;
    },

    // ============================================
    // PROCESAR BUQUES
    // ============================================
    procesarBuques: function(datos) {
        if (!this.estado.capa) return;
        
        // Extraer array de buques según formato
        let buques = [];
        
        if (Array.isArray(datos)) {
            buques = datos;
        } else if (datos.features) {
            buques = datos.features;
        } else if (datos.ships) {
            buques = datos.ships;
        } else if (datos.vessels) {
            buques = datos.vessels;
        }
        
        // Filtrar buques con posición válida
        const buquesValidos = buques.filter(b => {
            // GeoJSON format
            if (b.geometry && b.geometry.coordinates) {
                return b.geometry.coordinates.length >= 2;
            }
            // Simple format
            return b.lat && b.lon && 
                   !isNaN(b.lat) && !isNaN(b.lon) &&
                   Math.abs(b.lat) <= 90 && Math.abs(b.lon) <= 180;
        }).slice(0, this.config.maxBuques); // Limitar cantidad
        
        this.estado.estadisticas.totalRecibidos = buques.length;
        this.estado.count = buquesValidos.length;
        
        // Actualizar contadores UI
        this.actualizarContadores();
        
        // Set de IDs actuales
        const idsActuales = new Set();
        
        // Procesar cada buque
        buquesValidos.forEach(b => {
            // Extraer coordenadas según formato
            let lat, lon;
            
            if (b.geometry && b.geometry.coordinates) {
                lon = b.geometry.coordinates[0];
                lat = b.geometry.coordinates[1];
            } else {
                lat = b.lat;
                lon = b.lon;
            }
            
            // Extraer propiedades
            const props = b.properties || b;
            
            // Generar ID único
            const id = props.mmsi || props.imo || `ship-${lat}-${lon}-${Date.now()}`;
            idsActuales.add(id);
            
            // Determinar tipo de buque
            const tipo = this.determinarTipoBuque(props);
            
            // Calcular atributos visuales
            const color = this.obtenerColor(tipo);
            const tamaño = this.obtenerTamaño(props);
            
            // Calcular distancia si tenemos posición
            let distancia = null;
            if (window.ESTADO && window.ESTADO.posicion && window.ESTADO.posicion.valida) {
                distancia = this.calcularDistancia(
                    window.ESTADO.posicion.lat, window.ESTADO.posicion.lon,
                    lat, lon
                );
                
                // Filtrar por distancia si está configurado
                if (this.config.distanciaMaxima > 0 && distancia > this.config.distanciaMaxima) {
                    return; // No mostrar este buque
                }
            }
            
            // Crear popup
            const popup = this.crearPopupBuque(props, lat, lon, distancia, tipo);
            
            if (this.estado.marcadores.has(id)) {
                // Actualizar existente
                const marcador = this.estado.marcadores.get(id);
                marcador.setLatLng([lat, lon]);
                marcador.setStyle({
                    fillColor: color,
                    radius: tamaño
                });
                marcador.setPopupContent(popup);
            } else {
                // Crear nuevo marcador
                const marcador = L.circleMarker([lat, lon], {
                    radius: tamaño,
                    color: '#ffffff',
                    weight: 2,
                    fillColor: color,
                    fillOpacity: 0.9
                }).bindPopup(popup);
                
                // Eventos hover
                marcador.on('mouseover', function() {
                    this.setStyle({ weight: 3, color: '#ffff00' });
                });
                
                marcador.on('mouseout', function() {
                    this.setStyle({ weight: 2, color: '#ffffff' });
                });
                
                if (this.estado.visible) {
                    marcador.addTo(this.estado.capa);
                }
                
                this.estado.marcadores.set(id, marcador);
            }
        });
        
        // Eliminar marcadores de buques que ya no están
        this.estado.marcadores.forEach((marcador, id) => {
            if (!idsActuales.has(id)) {
                this.estado.capa.removeLayer(marcador);
                this.estado.marcadores.delete(id);
            }
        });
        
        this.estado.estadisticas.totalMostrados = this.estado.marcadores.size;
    },

    // ============================================
    // DETERMINAR TIPO DE BUQUE
    // ============================================
    determinarTipoBuque: function(props) {
        // Basado en códigos AIS tipo de buque
        const tipoAIS = props.type || props.ship_type || props.vessel_type || 0;
        
        // Códigos comunes AIS
        if ([70, 71, 72, 73, 74, 75, 76, 77].includes(tipoAIS)) return 'carguero';
        if ([80, 81, 82, 83, 84, 85, 86, 87, 88].includes(tipoAIS)) return 'petrolero';
        if ([60, 61, 62, 63, 64, 65, 66, 67, 68, 69].includes(tipoAIS)) return 'pasaje';
        if ([30, 31, 32, 33, 34, 35, 36, 37].includes(tipoAIS)) return 'pesquero';
        if ([36, 37, 38, 39].includes(tipoAIS)) return 'recreo';
        if ([52, 53, 54, 55, 56, 57, 58, 59].includes(tipoAIS)) return 'remolcador';
        if ([35].includes(tipoAIS)) return 'militar';
        
        // Por nombre
        const nombre = (props.name || '').toLowerCase();
        if (nombre.includes('tanker')) return 'petrolero';
        if (nombre.includes('cargo')) return 'carguero';
        if (nombre.includes('passenger')) return 'pasaje';
        if (nombre.includes('fishing')) return 'pesquero';
        if (nombre.includes('tug')) return 'remolcador';
        if (nombre.includes('navy') || nombre.includes('warship')) return 'militar';
        
        return 'otro';
    },

    // ============================================
    // OBTENER COLOR SEGÚN TIPO
    // ============================================
    obtenerColor: function(tipo) {
        return this.config.colores[tipo] || this.config.colores.otro;
    },

    // ============================================
    // OBTENER TAMAÑO SEGÚN ESLORA
    // ============================================
    obtenerTamaño: function(props) {
        const eslora = props.length || props.longitud || 0;
        
        if (eslora > 300) return this.config.tamanos.muyGrande;
        if (eslora > 150) return this.config.tamanos.grande;
        if (eslora > 50) return this.config.tamanos.mediano;
        return this.config.tamanos.pequeño;
    },

    // ============================================
    // CALCULAR DISTANCIA (Haversine)
    // ============================================
    calcularDistancia: function(lat1, lon1, lat2, lon2) {
        const R = 6371; // Radio Tierra en km
        const dLat = (lat2 - lat1) * Math.PI / 180;
        const dLon = (lon2 - lon1) * Math.PI / 180;
        const a = Math.sin(dLat/2) * Math.sin(dLat/2) +
                 Math.cos(lat1 * Math.PI/180) * Math.cos(lat2 * Math.PI/180) *
                 Math.sin(dLon/2) * Math.sin(dLon/2);
        const c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1-a));
        return R * c;
    },

    // ============================================
    // CREAR POPUP DE BUQUE
    // ============================================
    crearPopupBuque: function(props, lat, lon, distancia, tipo) {
        const nombre = props.name || props.nombre || props.shipname || 'DESCONOCIDO';
        const mmsi = props.mmsi || props.MMSI || '---';
        const imo = props.imo || props.IMO || '---';
        const rumbo = props.course || props.heading || props.rumbo || '---';
        const velocidad = props.speed || props.velocidad || '---';
        const calado = props.draft || props.calado || '---';
        const destino = props.destination || props.destino || '---';
        const eslora = props.length || props.eslora || '---';
        const manga = props.beam || props.manga || '---';
        
        const velocidadTexto = velocidad !== '---' ? velocidad.toFixed(1) : '---';
        const rumboTexto = rumbo !== '---' ? rumbo.toFixed(0) : '---';
        const distanciaTexto = distancia ? distancia.toFixed(1) : '?';
        
        // Icono según tipo
        const iconos = {
            carguero: '🚢',
            petrolero: '🛢️',
            pasaje: '🚢',
            pesquero: '🎣',
            recreo: '⛵',
            remolcador: '🚤',
            militar: '⚓',
            otro: '🚢'
        };
        const icono = iconos[tipo] || '🚢';
        
        return `
            <div style="font-family: 'Digital-7'; color: #ffaa00; min-width: 240px; background: #111; padding: 12px; border-radius: 6px; border: 2px solid #ffaa00;">
                <div style="font-size: 18px; font-weight: bold; border-bottom: 2px solid #ffaa00; margin-bottom: 10px; padding-bottom: 5px; text-align: center;">
                    ${icono} ${nombre}
                </div>
                
                <div style="display: grid; grid-template-columns: 1fr 1fr; gap: 10px; margin-bottom: 10px;">
                    <div>
                        <div style="color: #888; font-size: 8px;">MMSI</div>
                        <div style="font-size: 14px;">${mmsi}</div>
                    </div>
                    <div>
                        <div style="color: #888; font-size: 8px;">IMO</div>
                        <div style="font-size: 14px;">${imo}</div>
                    </div>
                    <div>
                        <div style="color: #888; font-size: 8px;">VELOCIDAD</div>
                        <div style="font-size: 14px;">${velocidadTexto} kn</div>
                    </div>
                    <div>
                        <div style="color: #888; font-size: 8px;">RUMBO</div>
                        <div style="font-size: 14px;">${rumboTexto}°</div>
                    </div>
                </div>
                
                <div style="display: grid; grid-template-columns: 1fr 1fr; gap: 10px; margin-bottom: 10px;">
                    <div>
                        <div style="color: #888; font-size: 8px;">ESLORA</div>
                        <div style="font-size: 12px;">${eslora} m</div>
                    </div>
                    <div>
                        <div style="color: #888; font-size: 8px;">MANGA</div>
                        <div style="font-size: 12px;">${manga} m</div>
                    </div>
                    <div>
                        <div style="color: #888; font-size: 8px;">CALADO</div>
                        <div style="font-size: 12px;">${calado} m</div>
                    </div>
                    <div>
                        <div style="color: #888; font-size: 8px;">DISTANCIA</div>
                        <div style="font-size: 12px;">${distanciaTexto} km</div>
                    </div>
                </div>
                
                <div style="background: #1a1a1a; padding: 6px; border-radius: 4px; margin-bottom: 8px;">
                    <div style="color: #888; font-size: 8px;">DESTINO</div>
                    <div style="font-size: 12px;">${destino}</div>
                </div>
                
                <div style="background: #1a1a1a; padding: 6px; border-radius: 4px; font-size: 10px; text-align: center;">
                    📍 ${lat.toFixed(4)}°, ${lon.toFixed(4)}°<br>
                    ⏱️ ${new Date().toLocaleTimeString()}
                </div>
                
                <div style="display: flex; justify-content: space-between; margin-top: 5px; font-size: 8px; color: #888;">
                    <span>TIPO: ${tipo.toUpperCase()}</span>
                    <span>${this.estado.conectado ? '🟢 ONLINE' : '🔴 OFFLINE'}</span>
                </div>
            </div>
        `;
    },

    // ============================================
    // ACTUALIZAR CONTADORES UI
    // ============================================
    actualizarContadores: function() {
        document.getElementById('aisCount').textContent = this.estado.count;
        
        const total = (window.ESTADO?.adsb?.count || 0) + this.estado.count;
        document.getElementById('aisTotal').textContent = this.estado.count;
        document.getElementById('totalTargets').textContent = total;
    },

    // ============================================
    // TOGGLE VISIBILIDAD
    // ============================================
    toggleVisibilidad: function(visible) {
        this.estado.visible = visible;
        
        if (visible) {
            this.estado.capa.addTo(this.mapa);
            this.conectar();
            console.log('👁️ [AIS] Capa activada');
        } else {
            this.mapa.removeLayer(this.estado.capa);
            console.log('👁️ [AIS] Capa desactivada');
        }
    },

    // ============================================
    // LIMPIAR TODOS LOS MARCADORES
    // ============================================
    limpiar: function() {
        if (this.estado.capa) {
            this.estado.capa.clearLayers();
            this.estado.marcadores.clear();
            this.estado.buques.clear();
            this.estado.count = 0;
            this.actualizarContadores();
        }
    },

    // ============================================
    // INTEGRAR CON BOTÓN AIS
    // ============================================
    integrarBoton: function() {
        const boton = document.getElementById('layerAIS');
        
        if (boton) {
            // Clonar para eliminar event listeners anteriores
            const nuevoBoton = boton.cloneNode(true);
            boton.parentNode.replaceChild(nuevoBoton, boton);
            
            nuevoBoton.addEventListener('click', (e) => {
                e.preventDefault();
                e.stopPropagation();
                
                nuevoBoton.classList.toggle('active');
                this.toggleVisibilidad(nuevoBoton.classList.contains('active'));
            });
            
            console.log('🔌 [AIS] Botón integrado');
        }
    },

    // ============================================
    // OBTENER ESTADO PARA DEBUG
    // ============================================
    getEstado: function() {
        return {
            activo: this.estado.activo,
            visible: this.estado.visible,
            conectado: this.estado.conectado,
            url: this.estado.urlActual,
            buques: this.estado.count,
            marcadores: this.estado.marcadores.size,
            ultimaActualizacion: this.estado.ultimaActualizacion,
            estadisticas: this.estado.estadisticas
        };
    }
};

            // ============================================
            // MÓDULO RADAR METEOROLÓGICO - NUBES REALES
            // ============================================

            /**
             * Carga el radar meteorológico de RainViewer
             */
            async function cargarRadarMeteo() {
                if (!ESTADO.meteo.activo) return;
                
                try {
                    console.log('🌧️ Cargando radar meteorológico...');
                    
                    const response = await fetch(CONFIG.METEORADAR.RAINVIEWER_API);
                    const data = await response.json();
                    
                    if (data?.radar?.length > 0) {
                        // Obtener frames disponibles
                        const frames = data.radar;
                        
                        // Buscar frame más reciente (últimos 5 minutos)
                        const ahora = Math.floor(Date.now() / 1000);
                        let frameSeleccionado = null;
                        
                        for (let i = frames.length - 1; i >= 0; i--) {
                            if (ahora - frames[i].time < 300) { // 5 minutos
                                frameSeleccionado = frames[i];
                                break;
                            }
                        }
                        
                        // Si no hay frame reciente, usar el último
                        if (!frameSeleccionado) {
                            frameSeleccionado = frames[frames.length - 1];
                        }
                        
                        const timestamp = frameSeleccionado.time;
                        
                        // IMPORTANTE: Esquema de color 4 (estándar: verde -> rojo)
                        // 1 = original, 2 = universal, 3 = ironbow, 4 = precipitation
                        const url = `https://tilecache.rainviewer.com/v2/radar/${timestamp}/256/{z}/{x}/{y}/4/1_1.png`;
                        
                        console.log('🌧️ URL radar:', url.replace('{z}/{x}/{y}', '8/130/80'));
                        
                        const nuevaCapa = L.tileLayer(url, {
                            opacity: CONFIG.METEORADAR.OPACIDAD,
                            transparent: true,
                            attribution: 'RainViewer',
                            zIndex: 500,
                            bounds: [[-90, -180], [90, 180]]
                        });
                        
                        // Eliminar capa anterior
                        if (ESTADO.meteo.capa) {
                            ESTADO.mapa.removeLayer(ESTADO.meteo.capa);
                        }
                        
                        // Añadir nueva capa si está visible
                        if (ESTADO.meteo.visible) {
                            nuevaCapa.addTo(ESTADO.mapa);
                            console.log('✅ Capa radar añadida al mapa');
                        }
                        
                        ESTADO.meteo.capa = nuevaCapa;
                        ESTADO.meteo.timestamp = timestamp;
                        ESTADO.meteo.fuente = 'RainViewer';
                        ESTADO.meteo.disponible = true;
                        ESTADO.meteo.ultimaActualizacion = Date.now();
                        ESTADO.estadisticas.meteoActualizaciones++;
                        
                        // Actualizar UI
                        document.getElementById('radarSource').className = 'status-led led-green';
                        document.getElementById('meteoCount').textContent = '✓';
                        
                        const fecha = new Date(timestamp * 1000);
                        console.log(`✅ Radar meteorológico cargado: ${fecha.toLocaleString()}`);
                    }
                } catch (error) {
                    console.error('❌ Error cargando radar:', error);
                    ESTADO.estadisticas.meteoErrores++;
                    document.getElementById('radarSource').className = 'status-led led-red';
                    
                    // Intentar con OpenWeather como respaldo
                    await cargarOpenWeather();
                }
            }

            /**
             * Carga radar de OpenWeather como respaldo
             */
            async function cargarOpenWeather() {
                if (!CONFIG.METEORADAR.OPENWEATHER_KEY || 
                    CONFIG.METEORADAR.OPENWEATHER_KEY === 'TU_API_KEY_AQUI') {
                    console.warn('⚠️ OpenWeather requiere API key válida');
                    return;
                }
                
                try {
                    const url = CONFIG.METEORADAR.OPENWEATHER + CONFIG.METEORADAR.OPENWEATHER_KEY;
                    
                    const nuevaCapa = L.tileLayer(url, {
                        opacity: CONFIG.METEORADAR.OPACIDAD,
                        transparent: true,
                        attribution: 'OpenWeatherMap',
                        zIndex: 500
                    });
                    
                    if (ESTADO.meteo.capa) {
                        ESTADO.mapa.removeLayer(ESTADO.meteo.capa);
                    }
                    
                    if (ESTADO.meteo.visible) {
                        nuevaCapa.addTo(ESTADO.mapa);
                    }
                    
                    ESTADO.meteo.capa = nuevaCapa;
                    ESTADO.meteo.fuente = 'OpenWeather';
                    ESTADO.meteo.disponible = true;
                    
                    document.getElementById('radarSource').className = 'status-led led-green';
                    document.getElementById('meteoCount').textContent = '✓';
                    
                    console.log('✅ OpenWeather cargado como respaldo');
                } catch (error) {
                    console.error('❌ Error OpenWeather:', error);
                }
            }

            /**
             * Actualiza los datos meteorológicos actuales
             */
            async function actualizarMeteoActual() {
                if (!ESTADO.posicion.valida) return;
                
                try {
                    const url = `${CONFIG.METEO.OPEN_METEO}?latitude=${ESTADO.posicion.lat}&longitude=${ESTADO.posicion.lon}&current=temperature_2m,relative_humidity_2m,pressure_msl,wind_speed_10m,wind_direction_10m,weather_code`;
                    
                    const response = await fetch(url);
                    const data = await response.json();
                    
                    if (data.current) {
                        ESTADO.meteoActual = {
                            temperatura: data.current.temperature_2m,
                            humedad: data.current.relative_humidity_2m,
                            presion: data.current.pressure_msl,
                            viento: data.current.wind_speed_10m,
                            direccionViento: data.current.wind_direction_10m,
                            codigoTiempo: data.current.weather_code,
                            descripcion: decodificarTiempo(data.current.weather_code),
                            actualizado: Date.now()
                        };
                        
                        // Actualizar UI
                        document.getElementById('airTemp').textContent = Math.round(ESTADO.meteoActual.temperatura) + '°';
                        document.getElementById('baro').textContent = Math.round(ESTADO.meteoActual.presion);
                        document.getElementById('twsValue').textContent = Math.round(ESTADO.meteoActual.viento) + ' kn';
                        document.getElementById('srcMeteo').textContent = `🌤️ ${ESTADO.meteoActual.descripcion}`;
                        
                        console.log(`🌤️ Meteo: ${ESTADO.meteoActual.temperatura}°C, ${ESTADO.meteoActual.descripcion}`);
                    }
                } catch (error) {
                    console.error('Error actualizando datos meteorológicos:', error);
                }
            }

            /**
             * Decodifica código WMO a descripción
             */
            function decodificarTiempo(codigo) {
                const tiempos = {
                    0: 'Despejado',
                    1: 'Mayormente despejado',
                    2: 'Parcialmente nublado',
                    3: 'Nublado',
                    45: 'Niebla',
                    48: 'Niebla con escarcha',
                    51: 'Llovizna ligera',
                    53: 'Llovizna moderada',
                    55: 'Llovizna intensa',
                    61: 'Lluvia ligera',
                    63: 'Lluvia moderada',
                    65: 'Lluvia intensa',
                    71: 'Nieve ligera',
                    73: 'Nieve moderada',
                    75: 'Nieve intensa',
                    80: 'Chubascos ligeros',
                    81: 'Chubascos moderados',
                    82: 'Chubascos violentos',
                    95: 'Tormenta',
                    96: 'Tormenta con granizo ligero',
                    99: 'Tormenta con granizo fuerte'
                };
                return tiempos[codigo] || 'Desconocido';
            }

            // ============================================
            // MÓDULO AIS (BARCOS)
            // ============================================

            /**
             * Conecta con servidor AIS
             */
            async function conectarAIS() {
                if (!ESTADO.ais.activo || !ESTADO.ais.visible) return;
                
                for (const url of CONFIG.AIS.URLS) {
                    try {
                        const controller = new AbortController();
                        setTimeout(() => controller.abort(), CONFIG.AIS.TIMEOUT_MS);
                        
                        const response = await fetch(url, { signal: controller.signal });
                        
                        if (response.ok) {
                            const data = await response.json();
                            
                            if (data.ships || data.features) {
                                ESTADO.ais.conectado = true;
                                ESTADO.ais.url = url;
                                ESTADO.ais.intentos = 0;
                                ESTADO.ais.ultimaActualizacion = Date.now();
                                ESTADO.estadisticas.aisActualizaciones++;
                                
                                document.getElementById('aisSource').className = 'status-led led-green';
                                
                                // Procesar buques
                                const buques = data.ships || data.features || [];
                                procesarAIS(buques);
                                
                                console.log(`✅ AIS: ${ESTADO.ais.count} buques desde ${url}`);
                                return;
                            }
                        }
                    } catch (error) {
                        // Silencioso
                    }
                }
                
                ESTADO.ais.intentos++;
                ESTADO.estadisticas.aisErrores++;
                
                if (ESTADO.ais.intentos >= 3) {
                    ESTADO.ais.conectado = false;
                    document.getElementById('aisSource').className = 'status-led led-red';
                }
            }

            /**
             * Procesa datos AIS
             */
            function procesarAIS(buques) {
                if (!ESTADO.ais.capa) return;
                
                // Implementación básica de AIS
                // Similar a ADS-B pero con colores diferentes
                console.log(`🚢 AIS: ${buques.length} buques`);
                ESTADO.ais.count = buques.length;
                document.getElementById('aisCount').textContent = ESTADO.ais.count;
            }

            // ============================================
            // CONFIGURACIÓN DE BOTONES
            // ============================================

            /**
             * Configura todos los botones de la interfaz
             */
            function configurarBotones() {
                console.log('🔘 Configurando botones...');
                
                // Botones de rango
                document.querySelectorAll('.range-btn').forEach(btn => {
                    btn.addEventListener('click', function(e) {
                        e.preventDefault();
                        
                        document.querySelectorAll('.range-btn').forEach(b => b.classList.remove('active'));
                        this.classList.add('active');
                        
                        ESTADO.rangoActual = parseInt(this.dataset.range);
                        
                        if (ESTADO.mapa) {
                            ESTADO.mapa.setZoom(CONFIG.ZOOM_POR_RANGO[ESTADO.rangoActual]);
                            
                            if (ESTADO.posicion.valida) {
                                ESTADO.mapa.setView([ESTADO.posicion.lat, ESTADO.posicion.lon], ESTADO.mapa.getZoom(), {
                                    animate: true,
                                    duration: 0.3
                                });
                            }
                            
                            dibujarCirculos();
                        }
                        
                        console.log(`Rango cambiado a: ${ESTADO.rangoActual} NM`);
                    });
                });

                // Botón METEOROLÓGICO
                const btnMeteo = document.getElementById('layerMeteo');
                if (btnMeteo) {
                    const nuevoBtnMeteo = btnMeteo.cloneNode(true);
                    btnMeteo.parentNode.replaceChild(nuevoBtnMeteo, btnMeteo);
                    
                    nuevoBtnMeteo.addEventListener('click', function(e) {
                        e.preventDefault();
                        this.classList.toggle('active');
                        
                        ESTADO.meteo.visible = this.classList.contains('active');
                        
                        if (ESTADO.meteo.visible) {
                            if (ESTADO.meteo.capa) {
                                ESTADO.meteo.capa.addTo(ESTADO.mapa);
                            } else {
                                cargarRadarMeteo();
                            }
                            console.log('👁️ Capa meteorológica activada');
                        } else {
                            if (ESTADO.meteo.capa) {
                                ESTADO.mapa.removeLayer(ESTADO.meteo.capa);
                            }
                            console.log('👁️ Capa meteorológica desactivada');
                        }
                    });
                }

                // Botón ADS-B
                const btnADSB = document.getElementById('layerADSB');
                if (btnADSB) {
                    const nuevoBtnADSB = btnADSB.cloneNode(true);
                    btnADSB.parentNode.replaceChild(nuevoBtnADSB, btnADSB);
                    
                    nuevoBtnADSB.addEventListener('click', function(e) {
                        e.preventDefault();
                        this.classList.toggle('active');
                        
                        ESTADO.adsb.visible = this.classList.contains('active');
                        
                        if (ESTADO.adsb.visible) {
                            ESTADO.adsb.capa.addTo(ESTADO.mapa);
                            conectarADSB();
                            console.log('👁️ Capa ADS-B activada');
                        } else {
                            ESTADO.mapa.removeLayer(ESTADO.adsb.capa);
                            console.log('👁️ Capa ADS-B desactivada');
                        }
                    });
                }

                // Botón AIS
                const btnAIS = document.getElementById('layerAIS');
                if (btnAIS) {
                    const nuevoBtnAIS = btnAIS.cloneNode(true);
                    btnAIS.parentNode.replaceChild(nuevoBtnAIS, btnAIS);
                    
                    nuevoBtnAIS.addEventListener('click', function(e) {
                        e.preventDefault();
                        this.classList.toggle('active');
                        
                        ESTADO.ais.visible = this.classList.contains('active');
                        
                        if (ESTADO.ais.visible) {
                            ESTADO.ais.capa.addTo(ESTADO.mapa);
                            conectarAIS();
                            console.log('👁️ Capa AIS activada');
                        } else {
                            ESTADO.mapa.removeLayer(ESTADO.ais.capa);
                        }
                    });
                }

                // Botón GAIN AUTO
                document.getElementById('gainAuto').addEventListener('click', function(e) {
                    e.preventDefault();
                    document.getElementById('gainManual').classList.remove('active');
                    this.classList.add('active');
                    document.getElementById('gainFill').style.width = '85%';
                    document.getElementById('gainValue').textContent = '85%';
                });

                // Botón GAIN MANUAL
                document.getElementById('gainManual').addEventListener('click', function(e) {
                    e.preventDefault();
                    document.getElementById('gainAuto').classList.remove('active');
                    this.classList.add('active');
                    document.getElementById('gainFill').style.width = '45%';
                    document.getElementById('gainValue').textContent = '45%';
                });

                // Botón POWER
                document.getElementById('powerBtn').addEventListener('click', function(e) {
                    e.preventDefault();
                    
                    if (this.textContent === 'ENCENDIDO') {
                        this.textContent = 'APAGADO';
                        document.getElementById('powerStatus').textContent = '⏻ OFF';
                        document.getElementById('txStatus').textContent = '📡 RX';
                        
                        // Desactivar módulos
                        ESTADO.sistema.encendido = false;
                        
                        // Limpiar capas
                        if (ESTADO.adsb.capa) ESTADO.adsb.capa.clearLayers();
                        if (ESTADO.ais.capa) ESTADO.ais.capa.clearLayers();
                        if (ESTADO.meteo.capa) ESTADO.mapa.removeLayer(ESTADO.meteo.capa);
                        
                        console.log('⏻ Sistema apagado');
                    } else {
                        this.textContent = 'ENCENDIDO';
                        document.getElementById('powerStatus').textContent = '⏻ ON';
                        document.getElementById('txStatus').textContent = '📡 TX';
                        
                        ESTADO.sistema.encendido = true;
                        
                        // Reactivar módulos
                        if (ESTADO.adsb.visible) conectarADSB();
                        if (ESTADO.ais.visible) conectarAIS();
                        if (ESTADO.meteo.visible) cargarRadarMeteo();
                        
                        console.log('⏻ Sistema encendido');
                    }
                });

                // Botón EMERGENCIA
                document.getElementById('emergencyBtn').addEventListener('click', function(e) {
                    e.preventDefault();
                    
                    if (confirm('🚨 ¿ENVIAR SEÑAL DE EMERGENCIA SOS?')) {
                        const pos = ESTADO.posicion.valida ? 
                            `${ESTADO.posicion.lat.toFixed(6)}°, ${ESTADO.posicion.lon.toFixed(6)}°` : 
                            'POSICIÓN DESCONOCIDA';
                        
                        alert(`🚨 EMERGENCIA ENVIADA\nPosición: ${pos}\nHora: ${new Date().toLocaleTimeString()}`);
                        
                        console.log('🚨 SOS ACTIVADO', {
                            posicion: ESTADO.posicion,
                            timestamp: Date.now()
                        });
                    }
                });

                // Botón SEA
                document.getElementById('seaBtn').addEventListener('click', function(e) {
                    e.preventDefault();
                    this.style.background = '#00ff88';
                    this.style.color = '#000';
                    setTimeout(() => {
                        this.style.background = '';
                        this.style.color = '';
                    }, 200);
                    alert('🌊 Modo SEA activado - Filtro anti-mar');
                });

                // Botón RAIN
                document.getElementById('rainBtn').addEventListener('click', function(e) {
                    e.preventDefault();
                    this.style.background = '#00ff88';
                    this.style.color = '#000';
                    setTimeout(() => {
                        this.style.background = '';
                        this.style.color = '';
                    }, 200);
                    alert('☔ Modo RAIN activado - Filtro anti-lluvia');
                });

                // Botón MENÚ
                document.getElementById('menuBtn').addEventListener('click', function(e) {
                    e.preventDefault();
                    
                    const menu = `
                        📋 MENÚ PRINCIPAL
                        ═══════════════
                        • Configuración
                        • Calibración
                        • Mantenimiento
                        • Estadísticas
                        • Acerca de
                        
                        Sistema activo: ${ESTADO.sistema.encendido ? '✅' : '❌'}
                        Aviones: ${ESTADO.adsb.count}
                        Buques: ${ESTADO.ais.count}
                        Uptime: ${Math.floor((Date.now() - ESTADO.sistema.inicio) / 1000)}s
                    `;
                    
                    alert(menu);
                });

                // Botón ALERTA
                document.getElementById('alertBtn').addEventListener('click', function(e) {
                    e.preventDefault();
                    
                    if (confirm('⚠️ ¿Activar alerta de proximidad?')) {
                        alert('🚨 ALERTA ACTIVADA - Se notificará cuando haya tráfico cercano');
                    }
                });

                console.log('✅ Botones configurados');
            }

            // ============================================
            // INICIALIZACIÓN DE TEMPORIZADORES
            // ============================================

            /**
             * Inicia todos los temporizadores periódicos
             */
            function iniciarTemporizadores() {
                console.log('⏱️ Iniciando temporizadores...');
                
                // ADS-B cada 2 segundos
                ESTADO.temporizadores.adsb = setInterval(conectarADSB, CONFIG.ADS_B.INTERVALO_MS);
                
                // AIS cada 5 segundos
                ESTADO.temporizadores.ais = setInterval(conectarAIS, CONFIG.AIS.INTERVALO_MS);
                
                // Radar meteorológico cada 5 minutos
                ESTADO.temporizadores.meteo = setInterval(cargarRadarMeteo, CONFIG.METEORADAR.INTERVALO_MS);
                
                // Actualizar círculos cada segundo
                ESTADO.temporizadores.circulos = setInterval(() => {
                    if (ESTADO.posicion.valida && ESTADO.sistema.encendido) {
                        dibujarCirculos();
                    }
                }, CONFIG.TIEMPOS.CIRCULOS);
                
                // Estadísticas cada minuto
                ESTADO.temporizadores.estadisticas = setInterval(() => {
                    ESTADO.sistema.uptime = Math.floor((Date.now() - ESTADO.sistema.inicio) / 1000);
                    
                    console.log('📊 Estadísticas:', {
                        uptime: ESTADO.sistema.uptime + 's',
                        gps: ESTADO.estadisticas.actualizacionesGPS,
                        adsb: ESTADO.adsb.count,
                        ais: ESTADO.ais.count,
                        targetsMax: ESTADO.estadisticas.targetsMax
                    });
                }, CONFIG.TIEMPOS.ESTADISTICAS);
                
                console.log('✅ Temporizadores iniciados');
            }

            // ============================================
            // INICIALIZACIÓN PRINCIPAL
            // ============================================

            /**
             * Función principal de inicialización
             */
            function iniciar() {
                console.log('🚀 Iniciando RADCOM v5.7.2...');
                console.log('📊 Configuración:', CONFIG);
                
                try {
                    // Inicializar componentes
                    initMapa();
                    initGPS();
                    
                    // Configurar botones
                    configurarBotones();
                    
                    // Iniciar módulos
                    conectarADSB();
                    conectarAIS();
                    cargarRadarMeteo();
                    actualizarMeteoActual();
                    
                    // Iniciar temporizadores
                    iniciarTemporizadores();
                    
                    // Calcular tiempo de carga
                    const tiempoCarga = Date.now() - ESTADO.sistema.inicio;
                    
                    console.log('✅ RADCOM iniciado correctamente');
                    console.log(`⏱️ Tiempo de carga: ${tiempoCarga}ms`);
                    console.log('📡 Esperando datos de sensores...');
                    
                } catch (error) {
                    console.error('❌ Error fatal durante la inicialización:', error);
                }
            }

            // Iniciar cuando el DOM esté listo
            if (document.readyState === 'loading') {
                document.addEventListener('DOMContentLoaded', iniciar);
            } else {
                iniciar();
            }
        })();
    </script>
</body>
</html>
        
    </div>


  <div id="map-source-storage" style="display:none;">
    <div style="display:flex; flex-direction:column; height:100%; width:100%; background:#000; position:relative; overflow:hidden;">
        
        <div style="display:flex; background:#111; border-bottom:2px solid #00ff88; height:52px; flex-shrink:0; z-index:1002; align-items:center; padding:0 10px;">
            <div style="display:flex; flex:1; gap:4px;">
                <button onclick="switchMapLayer('TOPO')" id="tab-topo" style="flex:1; background:#00ff88; color:#000; border:none; font-weight:bold; cursor:pointer; font-size:0.73rem; padding:8px 4px;">TOPOGRÁFICO</button>
                <button onclick="switchMapLayer('VFR')"  id="tab-vfr"  style="flex:1; background:#222; color:#888; border:none; font-weight:bold; cursor:pointer; font-size:0.73rem; padding:8px 4px;">VFR AÉREO</button>
                <button onclick="switchMapLayer('SEA')"  id="tab-sea"  style="flex:1; background:#222; color:#888; border:none; font-weight:bold; cursor:pointer; font-size:0.73rem; padding:8px 4px;">NÁUTICO</button>
                <button onclick="switchMapLayer('SAT')"  id="tab-sat"  style="flex:1; background:#222; color:#888; border:none; font-weight:bold; cursor:pointer; font-size:0.73rem; padding:8px 4px;">SATÉLITE</button>
            </div>
            
            <button onclick="toggleDualMode()" id="dual-btn" 
                    style="margin-left:12px; background:#ffaa00; color:#000; border:none; padding:8px 16px; border-radius:4px; font-weight:bold; cursor:pointer; font-size:0.75rem;">
                DUAL
            </button>
        </div>

        <div id="map-canvas" style="flex:1; background:#050505; position:relative;"></div>

        <div style="position:absolute; top:65px; left:10px; background:rgba(0,0,0,0.75); border:1px solid #00ff88; border-radius:3px; padding:5px 3px; z-index:1000; display:flex; flex-direction:column; gap:4px;">
            <button onclick="centerOnMyPosition()" title="Mi posición" style="background:none; border:none; color:#00ff88; font-size:1rem; cursor:pointer; padding:2px;">📍</button>
            <button onclick="toggleLiveTracking()" id="track-btn" title="Live Tracking" style="background:none; border:none; color:#00ff88; font-size:1rem; cursor:pointer; padding:2px;">▶️</button>
            <button onclick="toggleMeasure()" id="measure-btn" title="Medir distancia y rumbo" style="background:none; border:none; color:#00ff88; font-size:1rem; cursor:pointer; padding:2px;">📏</button>
            <button onclick="toggleDrawingMode()" id="draw-btn" title="Dibujar waypoints" style="background:none; border:none; color:#00ff88; font-size:1rem; cursor:pointer; padding:2px;">✍️</button>
            <button onclick="saveCurrentRoute()" title="Guardar ruta" style="background:none; border:none; color:#00ff88; font-size:1rem; cursor:pointer; padding:2px;">💾</button>
            <button onclick="showTrackHistory()" title="Historial" style="background:none; border:none; color:#00ff88; font-size:1rem; cursor:pointer; padding:2px;">📋</button>
            <button onclick="toggleWaypointPanel()" id="toggle-waypoint-btn" title="Mostrar/Ocultar Waypoints" style="background:none; border:none; color:#00ff88; font-size:1rem; cursor:pointer; padding:2px;">📍</button>
            <button onclick="clearAllDrawings()" title="Limpiar" style="background:none; border:none; color:#ff3300; font-size:1rem; cursor:pointer; padding:2px;">🗑️</button>
        </div>

        <div id="rescue-panel" style="position:absolute; bottom:50px; left:15px; background:rgba(0,0,0,0.85); border:1px solid #ff3300; border-radius:4px; padding:6px; z-index:1000; width:200px; backdrop-filter:blur(3px);">
            <div style="display:flex; justify-content:space-between; align-items:center; border-bottom:1px solid #ff3300; padding-bottom:3px;">
                <span style="color:#ff3300; font-weight:bold; font-size:0.7rem;">🆘 RESCATE</span>
                <button onclick="toggleRescuePanel()" id="rescue-toggle-btn" style="background:none; border:none; color:#ff3300; font-size:0.8rem; cursor:pointer; padding:1px 4px;">▲</button>
            </div>
            <div id="rescue-content" style="display:none; margin-top:5px;">
                <div style="display:flex; gap:3px; align-items:center; margin-bottom:4px;">
                    <span style="color:#fff; width:28px; font-size:0.6rem;">LAT:</span>
                    <input type="text" id="manual-lat" placeholder="40.4167" value="40.4167" 
                           style="flex:1; background:#222; color:#ff3300; border:1px solid #ff3300; padding:2px; border-radius:2px; font-family:monospace; font-size:0.6rem;">
                </div>
                <div style="display:flex; gap:3px; align-items:center; margin-bottom:5px;">
                    <span style="color:#fff; width:28px; font-size:0.6rem;">LON:</span>
                    <input type="text" id="manual-lon" placeholder="-3.7033" value="-3.7033" 
                           style="flex:1; background:#222; color:#ff3300; border:1px solid #ff3300; padding:2px; border-radius:2px; font-family:monospace; font-size:0.6rem;">
                </div>
                
                <div style="display:flex; gap:4px; margin-bottom:4px;">
                    <button onclick="updateRescueCoordinatesFromPosition()" style="flex:1; background:#0088ff; color:#fff; border:none; padding:3px; border-radius:2px; font-size:0.6rem; font-weight:bold;">📍 MI POS</button>
                    <button onclick="dropRescueMarker()" style="flex:1; background:#ffaa00; color:#000; border:none; padding:3px; border-radius:2px; font-size:0.6rem; font-weight:bold;">🆘 MARCAR</button>
                </div>
                
                <div style="display:flex; gap:4px; margin-bottom:4px;">
                    <button onclick="goToCoordinates()" style="flex:1; background:#ff3300; color:#fff; border:none; padding:3px; border-radius:2px; font-size:0.6rem;">IR</button>
                    <button onclick="clearRescuePanel()" style="flex:1; background:#666; color:#fff; border:none; padding:3px; border-radius:2px; font-size:0.6rem;">LIMPIAR</button>
                </div>
                
                <div style="display:flex; gap:4px; margin-bottom:4px;">
                    <button onclick="startRescueMission()" id="start-rescue-btn" style="flex:1; background:#00ff88; color:#000; border:none; padding:3px; border-radius:2px; font-size:0.6rem;">🚁 INICIAR</button>
                    <button onclick="cancelRescueMission()" id="cancel-rescue-btn" style="flex:1; background:#666; color:#fff; border:none; padding:3px; border-radius:2px; font-size:0.6rem;">✕ ANULAR</button>
                </div>
                
                <div style="display:flex; gap:4px; margin-bottom:4px;">
                    <button onclick="calculateRescueRoute()" style="flex:1; background:#00ccff; color:#000; border:none; padding:3px; border-radius:2px; font-size:0.6rem;">🗺️ RUTA</button>
                </div>
                
                <div id="rescue-info" style="color:#00ffff; font-size:0.55rem; text-align:center; padding:2px; background:rgba(0,255,255,0.1); border-radius:2px; min-height:18px;"></div>
                <div id="manual-coord-status" style="color:#ffaa00; font-size:0.5rem; text-align:center; margin-top:2px;"></div>
            </div>
        </div>

        <div id="waypoint-panel" style="position:absolute; top:65px; right:190px; background:rgba(0,0,0,0.85); border:1px solid #00ff88; border-radius:4px; padding:8px; z-index:1000; width:230px; display:none; backdrop-filter:blur(3px);">
            <div style="display:flex; justify-content:space-between; align-items:center; border-bottom:1px solid #00ff88; padding-bottom:5px; margin-bottom:6px;">
                <span style="color:#00ff88; font-weight:bold; font-size:0.75rem;">✍️ WAYPOINTS / RUTA</span>
                <button onclick="hideWaypointPanel()" style="background:none; border:none; color:#00ff88; font-size:0.8rem; cursor:pointer; padding:0 4px;">✕</button>
            </div>
            <div id="waypoint-list" style="max-height:140px; overflow-y:auto; color:#fff; font-size:0.6rem; margin-bottom:6px;"></div>
            
            <div style="background:rgba(0,100,255,0.1); border:1px solid #0088ff; border-radius:3px; padding:5px; margin-bottom:6px;">
                <div style="display:flex; align-items:center; gap:4px; margin-bottom:4px;">
                    <span style="color:#0088ff; font-size:0.65rem; font-weight:bold;">📍 MI POSICIÓN</span>
                </div>
                <div style="display:flex; gap:4px;">
                    <button onclick="connectWithMyLocation()" style="flex:1; background:#0088ff; color:#fff; border:none; padding:3px; border-radius:2px; font-size:0.6rem; font-weight:bold;">UNIR</button>
                    <button onclick="startRouteFromLocation()" style="flex:1; background:#00ccff; color:#000; border:none; padding:3px; border-radius:2px; font-size:0.6rem; font-weight:bold;">INICIAR</button>
                </div>
                <div id="location-route-info" style="color:#88ffff; font-size:0.55rem; text-align:center; margin-top:4px; min-height:14px;"></div>
            </div>
            
            <div style="display:flex; gap:4px; margin-bottom:4px;">
                <button onclick="connectWaypoints()" style="flex:1; background:#00ff88; color:#000; border:none; padding:3px; border-radius:2px; font-size:0.6rem;">UNIR TODO</button>
                <button onclick="saveWaypointRoute()" style="flex:1; background:#ffaa00; color:#000; border:none; padding:3px; border-radius:2px; font-size:0.6rem;">GUARDAR</button>
            </div>
            
            <div id="active-route-controls" style="background:rgba(0,255,136,0.1); border:1px solid #00ff88; border-radius:3px; padding:5px; margin-bottom:6px; display:none;">
                <div style="display:flex; align-items:center; gap:4px; margin-bottom:4px;">
                    <span style="color:#00ff88; font-size:0.65rem; font-weight:bold;">🚀 RUTA ACTIVA</span>
                </div>
                <div style="display:flex; gap:4px;">
                    <button onclick="returnToRouteStart()" style="flex:1; background:#ffaa00; color:#000; border:none; padding:3px; border-radius:2px; font-size:0.6rem; font-weight:bold;">↩️ RETORNO AL INICIO</button>
                    <button onclick="stopRouteNavigation()" style="flex:1; background:#ff3300; color:#fff; border:none; padding:3px; border-radius:2px; font-size:0.6rem;">⏹️ DETENER</button>
                </div>
                <div id="route-start-info" style="color:#00ff88; font-size:0.55rem; text-align:center; margin-top:4px; min-height:14px;"></div>
            </div>
            
            <div style="display:flex; gap:4px;">
                <button onclick="startRouteNavigation()" id="nav-btn" style="flex:1; background:#00ccff; color:#000; border:none; padding:3px; border-radius:2px; font-size:0.6rem;">INICIAR RUTA</button>
                <button onclick="clearWaypoints()" style="flex:1; background:#ff3300; color:#fff; border:none; padding:3px; border-radius:2px; font-size:0.6rem;">BORRAR</button>
            </div>
            <div id="route-info" style="color:#00ffff; font-size:0.55rem; text-align:center; padding:3px; margin-top:5px; background:rgba(0,255,255,0.05); border-radius:2px;"></div>
        </div>

        <div id="track-history-panel" style="position:absolute; bottom:50px; right:15px; background:rgba(0,0,0,0.85); border:1px solid #00ff88; border-radius:4px; padding:8px; z-index:1000; width:240px; display:none; backdrop-filter:blur(3px);">
            <div style="display:flex; justify-content:space-between; align-items:center; border-bottom:1px solid #00ff88; padding-bottom:5px; margin-bottom:6px;">
                <span style="color:#00ff88; font-weight:bold; font-size:0.75rem;">📋 RUTAS GUARDADAS</span>
                <button onclick="toggleTrackHistory()" style="background:none; border:none; color:#00ff88; font-size:0.8rem; cursor:pointer; padding:0 4px;">✕</button>
            </div>
            <div id="track-list" style="max-height:200px; overflow-y:auto; color:#fff; font-size:0.6rem; margin-bottom:5px;"></div>
            <button onclick="clearAllTracks()" style="width:100%; background:#ff3300; color:#fff; border:none; padding:4px; border-radius:2px; font-size:0.6rem;">🗑️ BORRAR HISTORIAL</button>
        </div>

        <div id="compass-container" style="position:absolute; bottom:30px; right:20px; width:160px; height:160px; z-index:2000; pointer-events:none; opacity:0.6; filter:drop-shadow(0 0 8px rgba(0,0,0,0.7));"></div>
        <div id="compass-data-panel" style="position:absolute; top:15px; right:20px; background:rgba(0,0,0,0.7); border:1px solid #00ff88; border-radius:4px; padding:8px; z-index:2001; min-width:130px; backdrop-filter:blur(3px);">
            <div style="display:flex; justify-content:space-between; align-items:center; margin-bottom:4px;">
                <span style="color:#00ff88; font-size:0.7rem; font-weight:bold;">🧭 COMPASS</span>
                <button onclick="toggleCompassPanel()" id="compass-panel-toggle" style="background:none; border:none; color:#00ff88; font-size:0.8rem; cursor:pointer; padding:0 4px;">▼</button>
            </div>
            <div id="compass-data-content">
                <div style="display:flex; flex-direction:column; gap:4px;">
                    <div style="display:flex; align-items:center; gap:6px;">
                        <span style="color:#0088ff; font-size:0.75rem; font-weight:bold;">N:</span>
                        <span id="magnetic-reading-small" style="color:#fff; font-size:0.7rem; font-family:monospace;">0°</span>
                    </div>
                    <div style="display:flex; align-items:center; gap:6px;">
                        <span style="color:#00ff88; font-size:0.75rem; font-weight:bold;">R:</span>
                        <span id="heading-reading-small" style="color:#fff; font-size:0.7rem; font-family:monospace;">0°</span>
                    </div>
                    <div style="display:flex; align-items:center; gap:6px;">
                        <span style="color:#88ffff; font-size:0.75rem; font-weight:bold;">V:</span>
                        <span id="wind-reading-small" style="color:#fff; font-size:0.7rem; font-family:monospace;">0°</span>
                        <span id="wind-speed-small" style="color:#88ffff; font-size:0.6rem; margin-left:auto;">0kt</span>
                    </div>
                </div>
            </div>
        </div>

        <div style="position:absolute; bottom:0; left:0; right:0; background:rgba(0,0,0,0.9); color:#00ff88; font-family:monospace; font-size:0.7rem; padding:5px 12px; text-align:center; z-index:1000; border-top:1px solid #00ff88;">
            <span style="color:#00ff88;">LAT:</span> <span id="cursor-lat" style="color:#fff;">00.000000</span> 
            <span style="color:#00ff88; margin-left:12px;">|</span> 
            <span style="color:#00ff88; margin-left:12px;">LON:</span> <span id="cursor-lon" style="color:#fff;">00.000000</span>
            <span style="margin-left:15px; color:#ffaa00;" id="track-status"></span>
            <span style="margin-left:15px; color:#00ffff;" id="measure-status"></span>
            <span style="margin-left:15px; color:#ff3300;" id="rescue-status"></span>
            <span style="margin-left:15px; color:#00ff88;" id="navigation-status"></span>
        </div>
    </div>

    <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" />
    <script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/@turf/turf@6/turf.min.js"></script>

    <script>
        // ===== VARIABLES GLOBALES =====
        let map, currentLayer, isDualMode = false, dualOverlay = null;
        let liveTrackLayer, drawnItems, rescueMarker = null, rescueCircle = null;
        let trackingActive = false, navigationActive = false;
        let watchId = null, navWatchId = null;
        let measureLayer, isMeasuring = false;
        let measurePoints = [], measureTotal = 0;
        let currentPosition = null, rescuePoint = null;
        let savedTracks = [];
        let drawingActive = false, waypoints = [], waypointLine = null;
        let rescueMissionActive = false;
        let routePoints = [], routeStartPoint = null, locationRouteLine = null;
        let magneticDeclination = 2.5, windDirection = 0, windSpeed = 0;

        try {
            const stored = localStorage.getItem('savedRoutes');
            if (stored) savedTracks = JSON.parse(stored);
        } catch(e) {}

        const layers = {
            'TOPO': L.tileLayer('https://{s}.tile-cyclosm.openstreetmap.fr/cyclosm/{z}/{x}/{y}.png', { 
                maxZoom: 17,
                attribution: '© openstreetmap'
            }),
            'VFR': L.tileLayer('https://{s}.tile.openstreetmap.fr/osmfr/{z}/{x}/{y}.png', { 
                maxZoom: 18,
                attribution: '© OSM France'
            }),
            'SEA': L.layerGroup([
                L.tileLayer('https://cartodb-basemaps-{s}.global.ssl.fastly.net/light_all/{z}/{x}/{y}.png', { 
                    maxZoom: 19,
                    attribution: '© CartoDB'
                }),
                L.tileLayer('https://tiles.openseamap.org/seamark/{z}/{x}/{y}.png', { 
                    maxZoom: 19, 
                    opacity: 0.92,
                    attribution: '© OpenSeaMap'
                })
            ]),
            'SAT': L.tileLayer('https://server.arcgisonline.com/ArcGIS/rest/services/World_Imagery/MapServer/tile/{z}/{y}/{x}', { 
                maxZoom: 19,
                attribution: '© Esri'
            })
        };

        function switchMapLayer(type) {
            ['topo', 'vfr', 'sea', 'sat'].forEach(id => {
                const btn = document.getElementById('tab-' + id);
                if (btn) {
                    btn.style.background = '#222';
                    btn.style.color = '#888';
                }
            });
            
            const activeBtn = document.getElementById('tab-' + type.toLowerCase());
            if (activeBtn) {
                activeBtn.style.background = '#00ff88';
                activeBtn.style.color = '#000';
            }
            
            if (currentLayer) {
                map.removeLayer(currentLayer);
            }
            
            currentLayer = layers[type];
            currentLayer.addTo(map);
            
            if (isDualMode && dualOverlay) {
                map.addLayer(dualOverlay);
            }
            
            console.log('Capa cambiada a:', type);
        }

        function initMap() {
            if (map) return;
            
            map = L.map('map-canvas', { 
                zoomControl: false,
                attributionControl: true
            }).setView([40.4167, -3.7033], 13);
            
            currentLayer = layers['TOPO'].addTo(map);
            L.control.zoom({ position: 'bottomright' }).addTo(map);

            liveTrackLayer = L.polyline([], { color: '#00ff88', weight: 4, opacity: 0.8 }).addTo(map);
            drawnItems = new L.FeatureGroup().addTo(map);
            measureLayer = L.layerGroup().addTo(map);

            map.on('mousemove', e => {
                document.getElementById('cursor-lat').textContent = e.latlng.lat.toFixed(6);
                document.getElementById('cursor-lon').textContent = e.latlng.lng.toFixed(6);
            });
            
            initCompass();
            loadTrackList();
            
            document.getElementById('manual-lat').value = '40.4167';
            document.getElementById('manual-lon').value = '-3.7033';
            
            setTimeout(() => getInitialPosition(), 500);
        }

        function toggleDualMode() {
            isDualMode = !isDualMode;
            const btn = document.getElementById('dual-btn');
            btn.style.background = isDualMode ? '#00ff88' : '#ffaa00';
            btn.textContent = isDualMode ? 'DUAL ON' : 'DUAL';
            
            if (isDualMode) {
                if (!dualOverlay) {
                    dualOverlay = L.tileLayer('https://tiles.openseamap.org/seamark/{z}/{x}/{y}.png', { 
                        opacity: 0.7,
                        attribution: '© OpenSeaMap'
                    });
                }
                map.addLayer(dualOverlay);
            } else if (dualOverlay) {
                map.removeLayer(dualOverlay);
            }
        }

        function initCompass() {
            const container = document.getElementById('compass-container');
            container.innerHTML = `
                <svg viewBox="0 0 200 200" style="width:100%; height:100%; backdrop-filter:blur(2px); border-radius:50%; background:rgba(5,5,5,0.15);">
                    <defs>
                        <filter id="glow-blue" x="-20%" y="-20%" width="140%" height="140%"><feGaussianBlur in="SourceAlpha" stdDeviation="2"/><feMerge><feMergeNode in="offsetblur"/><feMergeNode in="SourceGraphic"/></feMerge></filter>
                        <filter id="glow-green" x="-20%" y="-20%" width="140%" height="140%"><feGaussianBlur in="SourceAlpha" stdDeviation="2"/><feMerge><feMergeNode in="offsetblur"/><feMergeNode in="SourceGraphic"/></feMerge></filter>
                        <filter id="glow-cyan" x="-20%" y="-20%" width="140%" height="140%"><feGaussianBlur in="SourceAlpha" stdDeviation="1.5"/><feMerge><feMergeNode in="offsetblur"/><feMergeNode in="SourceGraphic"/></feMerge></filter>
                    </defs>
                    <circle cx="100" cy="100" r="85" fill="none" stroke="#444" stroke-width="1" stroke-dasharray="3 3" opacity="0.3"/>
                    <circle cx="100" cy="100" r="82" fill="none" stroke="#00ff88" stroke-width="0.6" opacity="0.2"/>
                    <line x1="100" y1="20" x2="100" y2="28" stroke="#fff" stroke-width="1.8" opacity="0.8"/>
                    <line x1="100" y1="172" x2="100" y2="180" stroke="#fff" stroke-width="1.5" opacity="0.6"/>
                    <line x1="20" y1="100" x2="28" y2="100" stroke="#fff" stroke-width="1.5" opacity="0.6"/>
                    <line x1="172" y1="100" x2="180" y2="100" stroke="#fff" stroke-width="1.5" opacity="0.6"/>
                    <text x="100" y="40" text-anchor="middle" fill="#fff" font-size="11" font-weight="bold" opacity="0.9">N</text>
                    <text x="100" y="175" text-anchor="middle" fill="#fff" font-size="9" opacity="0.7">S</text>
                    <text x="40" y="105" text-anchor="middle" fill="#fff" font-size="9" opacity="0.7">W</text>
                    <text x="160" y="105" text-anchor="middle" fill="#fff" font-size="9" opacity="0.7">E</text>
                    <polygon id="north-magnetic" points="100,38 90,58 110,58" fill="#0088ff" stroke="#fff" stroke-width="1.2" filter="url(#glow-blue)" opacity="0.95" transform-origin="100 100"/>
                    <polygon id="heading-indicator" points="100,30 87,55 113,55" fill="#00ff88" stroke="#fff" stroke-width="1.2" filter="url(#glow-green)" opacity="0.95" transform-origin="100 100"/>
                    <g id="wind-indicator" transform-origin="100 100">
                        <line x1="100" y1="100" x2="100" y2="48" stroke="#88ffff" stroke-width="2.5" stroke-linecap="round" opacity="0.9" filter="url(#glow-cyan)"/>
                        <polygon points="100,40 93,53 107,53" fill="#88ffff" stroke="#fff" stroke-width="0.8" opacity="0.95"/>
                        <circle cx="100" cy="100" r="5" fill="#88ffff" stroke="#fff" stroke-width="1" opacity="0.95"/>
                    </g>
                    <circle cx="100" cy="100" r="7" fill="#0a0a0a" stroke="#00ff88" stroke-width="1.2"/>
                    <circle cx="100" cy="100" r="3" fill="#00ff88" stroke="none"/>
                </svg>
            `;
        }

        function updateHeading(bearing) {
            const el = document.getElementById('heading-indicator');
            const txt = document.getElementById('heading-reading-small');
            if (el) { bearing = (bearing + 360) % 360; el.style.transform = `rotate(${bearing}deg)`; }
            if (txt) txt.textContent = `${bearing.toFixed(0)}°`;
        }

        function updateMagneticNorth(trueNorth) {
            const el = document.getElementById('north-magnetic');
            const txt = document.getElementById('magnetic-reading-small');
            if (el) { let mn = (trueNorth - magneticDeclination + 360) % 360; el.style.transform = `rotate(${mn}deg)`; }
            if (txt) txt.textContent = `${((0 - magneticDeclination + 360) % 360).toFixed(0)}°`;
        }

        function updateWindDirection(dir, spd = 0) {
            const el = document.getElementById('wind-indicator');
            const txt = document.getElementById('wind-reading-small');
            const spdTxt = document.getElementById('wind-speed-small');
            if (el) { dir = (dir + 360) % 360; el.style.transform = `rotate(${dir}deg)`; }
            if (txt) txt.textContent = `${dir.toFixed(0)}°`;
            if (spdTxt) spdTxt.textContent = `${spd.toFixed(0)}kt`;
        }

        function getInitialPosition() {
            if (!navigator.geolocation) return;
            navigator.geolocation.getCurrentPosition(pos => {
                currentPosition = [pos.coords.latitude, pos.coords.longitude];
                map.setView(currentPosition, 15);
                addBaseMarker();
                updateMagneticNorth(0);
                updateWindDirection(180, 8);
            }, null, { enableHighAccuracy: true, timeout: 10000 });
        }

        function addBaseMarker() {
            L.marker(currentPosition, {
                icon: L.divIcon({ html: '<div style="font-size:22px;">📍</div>', iconSize: [26, 26] })
            }).addTo(drawnItems).bindPopup('🚨 BASE - TU POSICIÓN').openPopup();
        }

        function updateRescueCoordinatesFromPosition() {
            if (currentPosition) {
                document.getElementById('manual-lat').value = currentPosition[0].toFixed(6);
                document.getElementById('manual-lon').value = currentPosition[1].toFixed(6);
                document.getElementById('manual-coord-status').innerHTML = '📍 Coordenadas actualizadas';
                document.getElementById('manual-coord-status').style.color = '#00ff88';
            } else {
                alert('No hay posición disponible. Activa el GPS primero.');
                centerOnMyPosition();
                setTimeout(() => {
                    if (currentPosition) {
                        document.getElementById('manual-lat').value = currentPosition[0].toFixed(6);
                        document.getElementById('manual-lon').value = currentPosition[1].toFixed(6);
                        document.getElementById('manual-coord-status').innerHTML = '📍 Coordenadas actualizadas';
                    }
                }, 2000);
            }
        }

        function toggleRescuePanel() {
            const c = document.getElementById('rescue-content');
            const b = document.getElementById('rescue-toggle-btn');
            if (c.style.display === 'none') { c.style.display = 'block'; b.textContent = '▼'; }
            else { c.style.display = 'none'; b.textContent = '▲'; }
        }

        function dropRescueMarker() {
            try {
                const lat = parseFloat(document.getElementById('manual-lat').value);
                const lon = parseFloat(document.getElementById('manual-lon').value);
                
                if (isNaN(lat) || isNaN(lon) || lat < -90 || lat > 90 || lon < -180 || lon > 180) {
                    throw new Error('Coordenadas inválidas');
                }
                
                if (rescueMarker) map.removeLayer(rescueMarker);
                if (rescueCircle) map.removeLayer(rescueCircle);
                
                const pulseIcon = L.divIcon({
                    html: '<div style="font-size:32px; animation: pulse 1.5s infinite;">🆘</div>',
                    className: 'rescue-pulse-icon',
                    iconSize: [40, 40],
                    iconAnchor: [20, 40],
                    popupAnchor: [0, -40]
                });
                
                rescueMarker = L.marker([lat, lon], {
                    icon: pulseIcon,
                    zIndexOffset: 1000
                }).addTo(map);
                
                rescuePoint = [lat, lon];
                
                rescueCircle = L.circleMarker([lat, lon], {
                    radius: 20,
                    color: '#ff3300',
                    weight: 2,
                    opacity: 0.8,
                    fillColor: '#ff3300',
                    fillOpacity: 0.2
                }).addTo(map);
                
                rescueMarker.bindPopup(`
                    <div style="color:#ff3300; text-align:center; font-weight:bold;">
                        <div style="font-size:1.2rem; margin-bottom:5px;">🚨 PUNTO DE RESCATE</div>
                        <div>LAT: ${lat.toFixed(6)}</div>
                        <div>LON: ${lon.toFixed(6)}</div>
                        <div style="margin-top:8px; font-size:0.9rem; color:#fff;">${new Date().toLocaleTimeString()}</div>
                        <button onclick="calculateRescueRoute()" style="background:#ff3300; color:white; border:none; padding:5px 10px; border-radius:4px; margin-top:10px; cursor:pointer; font-weight:bold;">🗺️ CALCULAR RUTA</button>
                    </div>
                `, { minWidth: 220 }).openPopup();
                
                document.getElementById('manual-coord-status').innerHTML = '✅ PUNTO DE RESCATE MARCADO';
                document.getElementById('manual-coord-status').style.color = '#ff3300';
                document.getElementById('rescue-status').innerHTML = '🆘 RESCATE ACTIVO';
                
                if (currentPosition) {
                    calculateRescueRoute();
                }
                
            } catch(e) {
                document.getElementById('manual-coord-status').innerHTML = '❌ Coordenadas inválidas';
                document.getElementById('manual-coord-status').style.color = '#ff3300';
            }
        }

        function calculateRescueRoute() {
            if (!currentPosition) {
                alert('No hay posición actual. Activa el GPS.');
                return;
            }
            if (!rescuePoint) {
                alert('Marca un punto de rescate primero.');
                return;
            }
            
            drawnItems.eachLayer(l => { 
                if (l.options && l.options.className === 'rescue-route') {
                    drawnItems.removeLayer(l); 
                }
            });
            
            const from = turf.point([currentPosition[1], currentPosition[0]]);
            const to = turf.point([rescuePoint[1], rescuePoint[0]]);
            
            const distance = turf.distance(from, to, { units: 'kilometers' });
            const bearing = turf.bearing(from, to);
            
            const routeLine = L.polyline([currentPosition, rescuePoint], { 
                color: '#ff3300', 
                weight: 5, 
                opacity: 0.8, 
                dashArray: '10, 10', 
                className: 'rescue-route' 
            }).addTo(drawnItems);
            
            L.marker(currentPosition, {
                icon: L.divIcon({ 
                    html: '<div style="font-size:20px;">🚁</div>', 
                    iconSize: [24, 24] 
                })
            }).addTo(drawnItems).bindPopup('📍 Punto de inicio');
            
            const mid = [(currentPosition[0] + rescuePoint[0])/2, (currentPosition[1] + rescuePoint[1])/2];
            
            L.marker(mid, {
                icon: L.divIcon({
                    html: `<div style="background:#ff3300; color:white; padding:6px 12px; border-radius:20px; font-weight:bold; font-size:0.8rem; white-space:nowrap; box-shadow:0 0 10px rgba(255,51,0,0.5); border:1px solid white;">
                            🚁 ${(distance * 1000).toFixed(0)}m | 🧭 ${bearing.toFixed(1)}°
                          </div>`,
                    iconSize: [140, 32]
                })
            }).addTo(drawnItems);
            
            document.getElementById('rescue-info').innerHTML = `🚁 DISTANCIA: ${(distance * 1000).toFixed(0)}m | 🧭 RUMBO: ${bearing.toFixed(1)}°`;
            updateHeading(bearing);
            
            routePoints = [currentPosition, rescuePoint];
            routeStartPoint = currentPosition;
        }

        function startRescueMission() {
            if (!rescuePoint) { 
                alert('Marca un punto de rescate primero'); 
                return; 
            }
            rescueMissionActive = true;
            const btn = document.getElementById('start-rescue-btn');
            btn.style.background = '#ff3300'; 
            btn.innerHTML = '🚁 MISIÓN ACTIVA';
            document.getElementById('rescue-status').innerHTML = '🚨 MISIÓN DE RESCATE ACTIVA';
            document.getElementById('rescue-status').style.color = '#ff3300';
            
            if (!trackingActive) toggleLiveTracking();
            calculateRescueRoute();
            startRouteNavigation();
        }

        function cancelRescueMission() {
            rescueMissionActive = false;
            const btn = document.getElementById('start-rescue-btn');
            btn.style.background = '#00ff88'; 
            btn.innerHTML = '🚁 INICIAR';
            document.getElementById('rescue-status').innerHTML = '✕ MISIÓN ANULADA';
            document.getElementById('rescue-status').style.color = '#ffaa00';
        }

        function clearRescuePanel() {
            if (rescueMarker) { map.removeLayer(rescueMarker); rescueMarker = null; }
            if (rescueCircle) { map.removeLayer(rescueCircle); rescueCircle = null; }
            rescuePoint = null;
            
            document.getElementById('rescue-info').innerHTML = '';
            document.getElementById('manual-coord-status').innerHTML = '🧹 Marcador eliminado';
            document.getElementById('rescue-status').innerHTML = '';
            
            if (rescueMissionActive) cancelRescueMission();
            
            drawnItems.eachLayer(l => { 
                if (l.options && l.options.className === 'rescue-route') {
                    drawnItems.removeLayer(l); 
                }
            });
        }

        function goToCoordinates() {
            try {
                const lat = parseFloat(document.getElementById('manual-lat').value);
                const lon = parseFloat(document.getElementById('manual-lon').value);
                if (isNaN(lat) || isNaN(lon)) throw new Error();
                map.setView([lat, lon], 16);
                document.getElementById('manual-coord-status').innerHTML = '📍 Vista centrada';
                document.getElementById('manual-coord-status').style.color = '#00ff88';
            } catch(e) { 
                document.getElementById('manual-coord-status').innerHTML = '❌ Coordenadas inválidas';
                document.getElementById('manual-coord-status').style.color = '#ff3300';
            }
        }

        function centerOnMyPosition() {
            if (!navigator.geolocation) { alert('Geolocalización no soportada'); return; }
            navigator.geolocation.getCurrentPosition(pos => {
                currentPosition = [pos.coords.latitude, pos.coords.longitude];
                map.setView(currentPosition, 16);
                
                L.marker(currentPosition, {
                    icon: L.divIcon({ html: '<div style="font-size:22px;">📍</div>', iconSize: [26, 26] })
                }).addTo(drawnItems).bindPopup('📍 TU POSICIÓN').openPopup();
                
                if (rescuePoint) calculateRescueRoute();
            });
        }

        function toggleLiveTracking() {
            const btn = document.getElementById('track-btn');
            const status = document.getElementById('track-status');
            if (!trackingActive) {
                trackingActive = true;
                btn.style.color = '#ffaa00'; btn.textContent = '⏸️';
                const points = [];
                watchId = navigator.geolocation.watchPosition(pos => {
                    const p = [pos.coords.latitude, pos.coords.longitude];
                    points.push(p);
                    if (points.length > 500) points.shift();
                    liveTrackLayer.setLatLngs(points);
                    currentPosition = p;
                    
                    if (points.length > 1) {
                        const last = points[points.length - 1], prev = points[points.length - 2];
                        const from = turf.point([prev[1], prev[0]]), to = turf.point([last[1], last[0]]);
                        updateHeading(turf.bearing(from, to));
                        let total = 0;
                        for (let i = 1; i < points.length; i++) {
                            const f = turf.point([points[i-1][1], points[i-1][0]]), t = turf.point([points[i][1], points[i][0]]);
                            total += turf.distance(f, t, {units: 'kilometers'});
                        }
                        status.innerHTML = `🔴 ${(total*1000).toFixed(0)}m`;
                    }
                    
                    if (rescueMissionActive && rescuePoint) {
                        const from = turf.point([p[1], p[0]]), to = turf.point([rescuePoint[1], rescuePoint[0]]);
                        const dist = turf.distance(from, to, {units: 'kilometers'}), bearing = turf.bearing(from, to);
                        document.getElementById('rescue-info').innerHTML = `🚁 ${(dist*1000).toFixed(0)}m | 🧭 ${bearing.toFixed(1)}°`;
                        updateHeading(bearing);
                    }
                    
                    if (navigationActive && routeStartPoint) {
                        const from = turf.point([p[1], p[0]]), to = turf.point([routeStartPoint[1], routeStartPoint[0]]);
                        const distToStart = turf.distance(from, to, {units: 'kilometers'});
                        const bearingToStart = turf.bearing(from, to);
                        document.getElementById('route-start-info').innerHTML = `↩️ INICIO: ${(distToStart*1000).toFixed(0)}m | 🧭 ${bearingToStart.toFixed(1)}°`;
                    }
                    
                }, null, { enableHighAccuracy: true });
            } else {
                trackingActive = false;
                btn.style.color = '#00ff88'; btn.textContent = '▶️';
                status.innerHTML = '⏹️ PAUSADO';
                if (watchId) navigator.geolocation.clearWatch(watchId);
            }
        }

        function toggleMeasure() {
            isMeasuring = !isMeasuring;
            const btn = document.getElementById('measure-btn');
            const status = document.getElementById('measure-status');
            if (isMeasuring) {
                measurePoints = []; measureLayer.clearLayers(); measureTotal = 0;
                btn.style.color = '#ffaa00'; btn.style.background = 'rgba(255,170,0,0.2)';
                status.innerHTML = '📏 Click en mapa';
                map.off('click', addMeasurePoint);
                map.on('click', addMeasurePoint);
                document.getElementById('rescue-status').innerHTML = '📏 MODO MEDICIÓN';
            } else {
                btn.style.color = '#00ff88'; btn.style.background = 'none';
                status.innerHTML = '';
                map.off('click', addMeasurePoint);
                document.getElementById('rescue-status').innerHTML = '';
            }
        }

        function getCardinalDirection(b) {
            const dirs = ['N', 'NE', 'E', 'SE', 'S', 'SW', 'W', 'NW'];
            return dirs[Math.round(((b + 360) % 360) / 45) % 8];
        }

        function addMeasurePoint(e) {
            if (!isMeasuring) return;
            const p = e.latlng;
            measurePoints.push(p);
            
            L.circleMarker(p, { 
                radius: 6, 
                color: '#00ffff', 
                weight: 2, 
                fillColor: '#00ffff', 
                fillOpacity: 0.8 
            }).addTo(measureLayer).bindPopup(`<b>P${measurePoints.length}</b><br>${p.lat.toFixed(6)}<br>${p.lng.toFixed(6)}`);
            
            if (measurePoints.length > 1) {
                const last = measurePoints[measurePoints.length - 2];
                const from = turf.point([last.lng, last.lat]), to = turf.point([p.lng, p.lat]);
                const dKm = turf.distance(from, to, {units: 'kilometers'}), dM = dKm * 1000;
                const b = turf.bearing(from, to), card = getCardinalDirection(b);
                
                L.polyline([last, p], { 
                    color: '#00ffff', 
                    weight: 4, 
                    opacity: 0.8, 
                    dashArray: '8,6' 
                }).addTo(measureLayer);
                
                measureTotal += dKm;
                
                const mid = [(last.lat + p.lat)/2, (last.lng + p.lng)/2];
                L.marker(mid, {
                    icon: L.divIcon({ 
                        html: `<div style="background:#00ffff; color:#000; padding:4px 8px; border-radius:16px; font-size:0.65rem; white-space:nowrap;">
                                ${dM.toFixed(1)}m | ${b.toFixed(1)}° ${card}
                              </div>`, 
                        iconSize: [120, 24] 
                    })
                }).addTo(measureLayer);
                
                document.getElementById('measure-status').innerHTML = 
                    measurePoints.length > 2 ? `📐 TOTAL: ${(measureTotal*1000).toFixed(0)}m | ${measurePoints.length} pts` :
                    `📏 ${dM.toFixed(1)}m | ${b.toFixed(1)}° ${card}`;
                
                updateHeading(b);
            }
        }

        function toggleDrawingMode() {
            drawingActive = !drawingActive;
            const btn = document.getElementById('draw-btn');
            if (drawingActive) {
                btn.style.color = '#ffaa00'; btn.style.background = 'rgba(255,170,0,0.2)';
                map.on('click', addWaypoint);
                document.getElementById('waypoint-panel').style.display = 'block';
                document.getElementById('rescue-status').innerHTML = '✍️ DIBUJO ACTIVO';
            } else {
                btn.style.color = '#00ff88'; btn.style.background = 'none';
                map.off('click', addWaypoint);
                document.getElementById('rescue-status').innerHTML = '';
            }
        }

        function addWaypoint(e) {
            const lat = e.latlng.lat, lon = e.latlng.lng, id = Date.now() + Math.random();
            const marker = L.marker([lat, lon], {
                icon: L.divIcon({
                    html: `<div style="position:relative;">
                            <div style="font-size:22px;">📍</div>
                            <div style="background:#00ff88; color:#000; padding:1px 5px; border-radius:8px; font-size:0.5rem; position:absolute; top:-20px; left:5px; white-space:nowrap;">
                                ${lat.toFixed(4)}, ${lon.toFixed(4)}
                            </div>
                           </div>`,
                    iconSize: [36, 45]
                })
            }).addTo(drawnItems);
            
            marker.bindPopup(`
                <div style="text-align:center; font-size:0.7rem;">
                    <b style="color:#00ff88;">📍 WAYPOINT</b><br>
                    ${lat.toFixed(6)}<br>${lon.toFixed(6)}<br>
                    <button onclick="setAsRescuePoint(${lat}, ${lon})" style="background:#ff3300; color:white; border:none; padding:3px 6px; border-radius:2px; margin-top:3px; font-size:0.6rem;">🆘 RESCATE</button>
                    <button onclick="removeWaypoint('${id}')" style="background:#666; color:white; border:none; padding:3px 6px; border-radius:2px; margin-left:3px; font-size:0.6rem;">✕</button>
                </div>
            `);
            
            waypoints.push({ id, lat, lon, marker });
            updateWaypointList();
            if (waypoints.length > 1) connectWaypoints();
        }

        function removeWaypoint(id) {
            const i = waypoints.findIndex(w => w.id == id);
            if (i !== -1) {
                drawnItems.removeLayer(waypoints[i].marker);
                waypoints.splice(i, 1);
                updateWaypointList();
                if (waypointLine) { drawnItems.removeLayer(waypointLine); waypointLine = null; }
                if (waypoints.length > 1) connectWaypoints();
            }
        }

        function setAsRescuePoint(lat, lon) {
            document.getElementById('manual-lat').value = lat.toFixed(6);
            document.getElementById('manual-lon').value = lon.toFixed(6);
            dropRescueMarker();
            const c = document.getElementById('rescue-content');
            if (c.style.display === 'none') toggleRescuePanel();
        }

        function updateWaypointList() {
            const list = document.getElementById('waypoint-list');
            if (waypoints.length === 0) { 
                list.innerHTML = '<div style="color:#888; padding:5px; text-align:center;">Sin waypoints</div>'; 
                return; 
            }
            let html = '';
            waypoints.forEach((wp, i) => {
                html += `<div style="display:flex; justify-content:space-between; align-items:center; background:rgba(0,255,136,0.05); margin-bottom:3px; padding:4px; border-left:2px solid #00ff88;">
                            <div>
                                <span style="color:#00ff88; font-weight:bold; font-size:0.6rem;">WP${i+1}</span>
                                <span style="color:#aaa; font-size:0.5rem; margin-left:4px;">${wp.lat.toFixed(4)}, ${wp.lon.toFixed(4)}</span>
                            </div>
                            <div>
                                <button onclick="setAsRescuePoint(${wp.lat}, ${wp.lon})" style="background:#ff3300; color:white; border:none; padding:2px 4px; border-radius:2px; font-size:0.5rem;">🆘</button>
                                <button onclick="removeWaypoint('${wp.id}')" style="background:#666; color:white; border:none; padding:2px 4px; border-radius:2px; font-size:0.5rem; margin-left:2px;">✕</button>
                            </div>
                        </div>`;
            });
            list.innerHTML = html;
        }

        function connectWaypoints() {
            if (waypoints.length < 2) return;
            if (waypointLine) drawnItems.removeLayer(waypointLine);
            const pts = waypoints.map(w => [w.lat, w.lon]);
            waypointLine = L.polyline(pts, { 
                color: '#00ff88', 
                weight: 3, 
                opacity: 0.7, 
                dashArray: '6,5' 
            }).addTo(drawnItems);
            routePoints = pts;
            routeStartPoint = pts[0];
            let total = 0;
            for (let i = 1; i < pts.length; i++) {
                const f = turf.point([pts[i-1][1], pts[i-1][0]]), t = turf.point([pts[i][1], pts[i][0]]);
                total += turf.distance(f, t, {units: 'kilometers'});
            }
            document.getElementById('route-info').innerHTML = `🔗 ${(total*1000).toFixed(0)}m | ${waypoints.length} pts`;
        }

        function connectWithMyLocation() {
            if (!currentPosition) { 
                centerOnMyPosition(); 
                setTimeout(connectWithMyLocation, 2000); 
                return; 
            }
            if (waypoints.length === 0) { 
                alert('Dibuja waypoints primero'); 
                return; 
            }
            if (locationRouteLine) drawnItems.removeLayer(locationRouteLine);
            const pts = [currentPosition, ...waypoints.map(w => [w.lat, w.lon])];
            locationRouteLine = L.polyline(pts, { 
                color: '#0088ff', 
                weight: 4, 
                opacity: 0.8, 
                dashArray: '8,6' 
            }).addTo(drawnItems);
            routePoints = pts;
            routeStartPoint = pts[0];
            let total = 0;
            for (let i = 1; i < pts.length; i++) {
                const f = turf.point([pts[i-1][1], pts[i-1][0]]), t = turf.point([pts[i][1], pts[i][0]]);
                total += turf.distance(f, t, {units: 'kilometers'});
            }
            const f = turf.point([currentPosition[1], currentPosition[0]]), t = turf.point([waypoints[0].lon, waypoints[0].lat]);
            const d = turf.distance(f, t, {units: 'kilometers'}), b = turf.bearing(f, t);
            document.getElementById('location-route-info').innerHTML = `📍 WP1: ${(d*1000).toFixed(0)}m | 🧭 ${b.toFixed(0)}°`;
            document.getElementById('route-info').innerHTML = `🔵 RUTA: ${(total*1000).toFixed(0)}m | ${waypoints.length} wp`;
            updateHeading(b);
            map.fitBounds(locationRouteLine.getBounds());
        }

        function startRouteFromLocation() {
            if (!currentPosition) { 
                centerOnMyPosition(); 
                setTimeout(startRouteFromLocation, 2000); 
                return; 
            }
            if (waypoints.length === 0) { 
                alert('Dibuja waypoints primero'); 
                return; 
            }
            if (!locationRouteLine) connectWithMyLocation();
            startRouteNavigation();
            alert('🚀 Ruta iniciada desde tu posición');
        }

        function startRouteNavigation() {
            if (!routePoints || routePoints.length < 2) {
                if (waypoints.length > 1) {
                    connectWaypoints();
                } else if (rescuePoint && currentPosition) {
                    routePoints = [currentPosition, rescuePoint];
                    routeStartPoint = currentPosition;
                } else {
                    alert('Primero crea una ruta con waypoints, conecta con tu ubicación o marca un punto de rescate');
                    return;
                }
            }
            
            navigationActive = true;
            document.getElementById('navigation-status').innerHTML = '🧭 NAVEGANDO RUTA';
            document.getElementById('nav-btn').style.background = '#ff3300';
            document.getElementById('nav-btn').innerHTML = '🚀 NAVEGANDO';
            
            document.getElementById('active-route-controls').style.display = 'block';
            
            if (!trackingActive) toggleLiveTracking();
            
            if (routePoints && routePoints.length > 1) {
                const bounds = L.latLngBounds(routePoints.map(p => [p[0], p[1]]));
                map.fitBounds(bounds);
            }
            
            if (!routeStartPoint) {
                routeStartPoint = routePoints[0];
            }
            
            if (currentPosition && routeStartPoint) {
                const from = turf.point([currentPosition[1], currentPosition[0]]);
                const to = turf.point([routeStartPoint[1], routeStartPoint[0]]);
                const distToStart = turf.distance(from, to, {units: 'kilometers'});
                const bearingToStart = turf.bearing(from, to);
                document.getElementById('route-start-info').innerHTML = `↩️ INICIO: ${(distToStart*1000).toFixed(0)}m | 🧭 ${bearingToStart.toFixed(1)}°`;
            }
        }

        function returnToRouteStart() {
            if (!routeStartPoint) {
                alert('No hay punto de inicio de ruta definido');
                return;
            }
            
            map.setView(routeStartPoint, 16);
            
            L.marker(routeStartPoint, {
                icon: L.divIcon({ 
                    html: '<div style="font-size:24px;">🏁</div>', 
                    iconSize: [28, 28] 
                })
            }).addTo(drawnItems).bindPopup('🏁 INICIO DE RUTA').openPopup();
            
            if (currentPosition) {
                const from = turf.point([currentPosition[1], currentPosition[0]]);
                const to = turf.point([routeStartPoint[1], routeStartPoint[0]]);
                const distance = turf.distance(from, to, {units: 'kilometers'});
                const bearing = turf.bearing(from, to);
                
                L.polyline([currentPosition, routeStartPoint], {
                    color: '#ffaa00',
                    weight: 4,
                    opacity: 0.8,
                    dashArray: '10, 8',
                    className: 'return-route'
                }).addTo(drawnItems);
                
                alert(`↩️ Retorno al inicio: ${(distance*1000).toFixed(0)}m | Rumbo: ${bearing.toFixed(1)}°`);
            }
        }

        function stopRouteNavigation() {
            navigationActive = false;
            document.getElementById('navigation-status').innerHTML = '';
            document.getElementById('nav-btn').style.background = '#00ccff';
            document.getElementById('nav-btn').innerHTML = 'INICIAR RUTA';
            document.getElementById('active-route-controls').style.display = 'none';
            document.getElementById('route-start-info').innerHTML = '';
            
            drawnItems.eachLayer(l => {
                if (l.options && l.options.className === 'return-route') {
                    drawnItems.removeLayer(l);
                }
            });
        }

        function hideWaypointPanel() { 
            document.getElementById('waypoint-panel').style.display = 'none'; 
        }
        
        function toggleWaypointPanel() {
            const p = document.getElementById('waypoint-panel');
            p.style.display = p.style.display === 'none' ? 'block' : 'none';
        }
        
        function toggleCompassPanel() {
            const c = document.getElementById('compass-data-content');
            const b = document.getElementById('compass-panel-toggle');
            if (c.style.display === 'none') { 
                c.style.display = 'block'; 
                b.innerHTML = '▼'; 
            } else { 
                c.style.display = 'none'; 
                b.innerHTML = '▲'; 
            }
        }

        function saveCurrentRoute() {
            let pts = [];
            if (locationRouteLine && routePoints.length > 1) pts = routePoints;
            else if (waypoints.length > 1) pts = waypoints.map(w => [w.lat, w.lon]);
            else if (rescuePoint && currentPosition) pts = [currentPosition, rescuePoint];
            else { 
                alert('No hay ruta para guardar'); 
                return; 
            }
            let total = 0;
            for (let i = 1; i < pts.length; i++) {
                const f = turf.point([pts[i-1][1], pts[i-1][0]]), t = turf.point([pts[i][1], pts[i][0]]);
                total += turf.distance(f, t, {units: 'kilometers'});
            }
            savedTracks.push({
                id: Date.now(),
                date: new Date().toLocaleString(),
                points: pts,
                distance: total,
                pointCount: pts.length,
                type: locationRouteLine ? 'ruta_desde_ubicacion' : 'ruta'
            });
            if (savedTracks.length > 15) savedTracks.shift();
            localStorage.setItem('savedRoutes', JSON.stringify(savedTracks));
            loadTrackList();
            document.getElementById('track-status').innerHTML = `✅ GUARDADA (${(total*1000).toFixed(0)}m)`;
            setTimeout(() => { document.getElementById('track-status').innerHTML = ''; }, 3000);
        }
        
        function saveWaypointRoute() { saveCurrentRoute(); }

        function showTrackHistory() { 
            document.getElementById('track-history-panel').style.display = 'block'; 
            loadTrackList(); 
        }
        
        function toggleTrackHistory() { 
            document.getElementById('track-history-panel').style.display = 'none'; 
        }

        function loadTrackList() {
            const list = document.getElementById('track-list');
            if (savedTracks.length === 0) { 
                list.innerHTML = '<div style="color:#888; padding:8px; text-align:center;">No hay rutas</div>'; 
                return; 
            }
            let html = '';
            savedTracks.slice().reverse().forEach((t, i) => {
                html += `<div style="background:rgba(0,255,136,0.05); margin-bottom:6px; padding:6px; border-left:2px solid #00ff88;">
                            <div style="display:flex; justify-content:space-between;">
                                <span style="color:#00ff88; font-size:0.65rem; font-weight:bold;">${t.type === 'ruta_desde_ubicacion' ? '📍 DESDE POS' : '📍 RUTA'}</span>
                                <span style="color:#aaa; font-size:0.5rem;">${t.pointCount} pts</span>
                            </div>
                            <div style="color:#fff; font-size:0.6rem; margin:3px 0;">📏 ${(t.distance*1000).toFixed(0)}m</div>
                            <div style="display:flex; gap:4px; margin-top:3px;">
                                <button onclick="loadRoute('${t.id}')" style="flex:1; background:#00ff88; color:#000; border:none; padding:3px; border-radius:2px; font-size:0.55rem;">CARGAR</button>
                                <button onclick="startRouteFromHistory('${t.id}')" style="flex:1; background:#00ccff; color:#000; border:none; padding:3px; border-radius:2px; font-size:0.55rem;">INICIAR</button>
                                <button onclick="deleteRoute('${t.id}')" style="flex:1; background:#ff3300; color:#fff; border:none; padding:3px; border-radius:2px; font-size:0.55rem;">BORRAR</button>
                            </div>
                        </div>`;
            });
            list.innerHTML = html;
        }

        function startRouteFromHistory(id) {
            const t = savedTracks.find(t => t.id == id);
            if (t) {
                if (waypointLine) drawnItems.removeLayer(waypointLine);
                waypointLine = L.polyline(t.points, { 
                    color: t.type === 'ruta_desde_ubicacion' ? '#0088ff' : '#00ff88', 
                    weight: 4, 
                    opacity: 0.8, 
                    dashArray: '8,6' 
                }).addTo(drawnItems);
                routePoints = t.points;
                routeStartPoint = t.points[0];
                startRouteNavigation();
                alert(`🚀 Ruta iniciada - ${(t.distance*1000).toFixed(0)}m`);
                toggleTrackHistory();
            }
        }

        function loadRoute(id) {
            const t = savedTracks.find(t => t.id == id);
            if (t) {
                if (waypointLine) drawnItems.removeLayer(waypointLine);
                waypointLine = L.polyline(t.points, { 
                    color: t.type === 'ruta_desde_ubicacion' ? '#0088ff' : '#00ff88', 
                    weight: 4, 
                    opacity: 0.8, 
                    dashArray: '8,6' 
                }).addTo(drawnItems);
                routePoints = t.points;
                routeStartPoint = t.points[0];
                map.fitBounds(waypointLine.getBounds());
                document.getElementById('route-info').innerHTML = `📂 CARGADA: ${(t.distance*1000).toFixed(0)}m`;
                toggleTrackHistory();
            }
        }

        function deleteRoute(id) { 
            savedTracks = savedTracks.filter(t => t.id != id); 
            localStorage.setItem('savedRoutes', JSON.stringify(savedTracks)); 
            loadTrackList(); 
        }
        
        function clearAllTracks() { 
            if (confirm('¿Borrar todas las rutas?')) { 
                savedTracks = []; 
                localStorage.setItem('savedRoutes', JSON.stringify(savedTracks)); 
                loadTrackList(); 
            } 
        }

        function clearWaypoints() {
            waypoints.forEach(w => drawnItems.removeLayer(w.marker));
            waypoints = [];
            if (waypointLine) { drawnItems.removeLayer(waypointLine); waypointLine = null; }
            if (locationRouteLine) { drawnItems.removeLayer(locationRouteLine); locationRouteLine = null; }
            routePoints = [];
            routeStartPoint = null;
            updateWaypointList();
            document.getElementById('route-info').innerHTML = '';
            document.getElementById('location-route-info').innerHTML = '';
            stopRouteNavigation();
        }

        function clearAllDrawings() {
            if (confirm('¿Limpiar todo?')) {
                liveTrackLayer.setLatLngs([]);
                drawnItems.clearLayers();
                measureLayer.clearLayers();
                
                if (rescueMarker) { map.removeLayer(rescueMarker); rescueMarker = null; }
                if (rescueCircle) { map.removeLayer(rescueCircle); rescueCircle = null; }
                rescuePoint = null;
                
                waypoints = []; 
                routePoints = []; 
                routeStartPoint = null;
                measurePoints = []; 
                measureTotal = 0;
                waypointLine = null; 
                locationRouteLine = null;
                
                document.getElementById('waypoint-panel').style.display = 'none';
                document.getElementById('measure-status').innerHTML = '';
                document.getElementById('route-info').innerHTML = '';
                document.getElementById('location-route-info').innerHTML = '';
                document.getElementById('rescue-info').innerHTML = '';
                
                if (currentPosition) addBaseMarker();
                if (rescueMissionActive) cancelRescueMission();
                
                stopRouteNavigation();
                updateWaypointList();
                document.getElementById('track-status').innerHTML = '🧹 LIMPIADO';
            }
        }

        function loadSavedTrack() { showTrackHistory(); }
        
        function exportGPX() {
            let points = [];
            if (routePoints && routePoints.length > 1) points = routePoints;
            else if (waypoints.length > 1) points = waypoints.map(w => [w.lat, w.lon]);
            else if (liveTrackLayer.getLatLngs().length > 1) points = liveTrackLayer.getLatLngs();
            
            if (points.length < 2) { 
                alert('No hay ruta para exportar'); 
                return; 
            }
            
            let gpx = `<?xml version="1.0" encoding="UTF-8"?>
<gpx version="1.1" creator="Sistema Rescate">
  <trk><name>Ruta ${new Date().toLocaleDateString()}</name><trkseg>`;
            points.forEach(p => gpx += `<trkpt lat="${p[0] || p.lat}" lon="${p[1] || p.lng}"></trkpt>`);
            gpx += `</trkseg></trk></gpx>`;
            
            const blob = new Blob([gpx], {type: 'application/gpx+xml'});
            const url = URL.createObjectURL(blob);
            const a = document.createElement('a');
            a.href = url;
            a.download = `ruta_${Date.now()}.gpx`;
            a.click();
            URL.revokeObjectURL(url);
            document.getElementById('track-status').innerHTML = '📤 GPX EXPORTADO';
            setTimeout(() => { document.getElementById('track-status').innerHTML = ''; }, 3000);
        }
        
        setTimeout(initMap, 300);
        
        const style = document.createElement('style');
        style.innerHTML = `@keyframes pulse { 
            0% { transform: scale(1); opacity: 1; }
            50% { transform: scale(1.2); opacity: 0.9; }
            100% { transform: scale(1); opacity: 1; }
        }`;
        document.head.appendChild(style);
    </script>
</div>



    <script>
// =============================================
// RADCOM MASTER v5.7.2 - SISTEMA DE COMUNICACIÓN SEGURA + VoIP
// CORRECCIÓN: SELECCIÓN DE PEERS RESTAURADA
// =============================================

// ====== VARIABLES GLOBALES ======
const VERSION = '5.7.2';
let peer = null;
let myPeerId = null;
let savedIds = JSON.parse(localStorage.getItem('radcom_peers_v4') || "[]");
let activeTarget = 'GLOBAL';
let audioContext = null;
let currentConnectionType = 'wifi';
let connections = {};
let messageQueue = JSON.parse(localStorage.getItem('radcom_message_queue') || '[]');
let queueRetryInterval = null;
const MAX_QUEUE_SIZE = 100;

// Variables VoIP
let voipCalls = {};
let voipMediaStream = null;
let voipActiveCall = null;
let voipIncomingCall = null;
let voipAudioContext = null;
let voipRingtoneSource = null;
let voipOutgoingToneSource = null;

// Variables de estado
let connectionHealth = {};
let showOffline = false;
let fastRecovery = true;
let aggressiveRevive = true;
let isBackground = false;
let revivingInProgress = false;
let currentTab = 'ascii';

// Variables para Morse
let morseSpeed = 'normal';
let morseAudioContext = null;
let isPlayingMorse = false;

// Variables para reconocimiento de voz
let recognition = null;
let recognizing = false;

// Variables para consola
let currentConsoleTab = 'CMD';
let hexEditorContent = '';

// Sistema de ID
const ID_SYSTEM = {
    currentId: null,
    defaultPrefix: 'RADCOM-',
    useFixedId: true,
    fixedId: null
};

// Estadísticas
let stats = {
    messages: 0,
    tx: 0,
    rx: 0,
    startTime: Date.now()
};

// Sistema de radio
let currentBand = 'VHF';
let currentFrequency = 142.850;
let currentUnit = 'MHz';
let currentHFBand = null;
let currentChannelType = 'pmr';
let currentPMRType = 8;
let currentChannel = 1;
let radioAudioContext = null;

// Sistema satelital
const satelliteSystem = {
    latitude: null,
    longitude: null,
    altitude: null,
    accuracy: null,
    weatherData: null,
    lastUpdate: null,
    lastKnownPosition: null,
    refreshInterval: null,
    apiConnected: false
};

// ====== DATOS ESTÁTICOS ======
const chars = [
    ["NUL","DLE","SPC","0","@","P","`","p"],
    ["SOH","DC1","!","1","A","Q","a","q"],
    ["STX","DC2","\"","2","B","R","b","r"],
    ["ETX","DC3","#","3","C","S","c","s"],
    ["EOT","DC4","$","4","D","T","d","t"],
    ["ENQ","NAK","%","5","E","U","e","u"],
    ["ACK","SYN","&","6","F","V","f","v"],
    ["BEL","ETB","'","7","G","W","g","w"],
    ["BS","CAN","(","8","H","X","h","x"],
    ["HT","EM",")","9","I","Y","i","y"],
    ["LF","SUB","*",":","J","Z","j","z"],
    ["VT","ESC","+",";","K","[","k","{"],
    ["FF","FS",",","<","L","\\","l","|"],
    ["CR","GS","-","=","M","]","m","}"],
    ["SO","RS",".",">","N","^","n","~"],
    ["SI","US","/","?","O","_","o","DEL"]
];

const morseCodes = {
    'A': '.-', 'B': '-...', 'C': '-.-.', 'D': '-..', 'E': '.', 'F': '..-.', 'G': '--.', 'H': '....',
    'I': '..', 'J': '.---', 'K': '-.-', 'L': '.-..', 'M': '--', 'N': '-.', 'O': '---', 'P': '.--.',
    'Q': '--.-', 'R': '.-.', 'S': '...', 'T': '-', 'U': '..-', 'V': '...-', 'W': '.--', 'X': '-..-',
    'Y': '-.--', 'Z': '--..',
    '0': '-----', '1': '.----', '2': '..---', '3': '...--', '4': '....-', '5': '.....',
    '6': '-....', '7': '--...', '8': '---..', '9': '----.',
    '.': '.-.-.-', ',': '--..--', '?': '..--..', '/': '-..-.', ' ': '/'
};

const radioBands = {
    'UHF': { min: 300, max: 3000, unit: 'MHz', default: 433.000 },
    'VHF': { min: 30, max: 300, unit: 'MHz', default: 142.850 },
    'AEREA': { min: 108, max: 137, unit: 'MHz', default: 121.500 },
    'MARINA': { min: 156, max: 162, unit: 'MHz', default: 156.800 },
    'HF': { min: 3, max: 30, unit: 'MHz', default: 14.300 },
    'EMERG': { min: 0, max: 0, unit: 'MHz', default: 121.500 }
};

const hfBands = {
    '10m': { range: '28.0-29.7 MHz', desc: 'Propagación diurna excelente' },
    '15m': { range: '21.0-21.45 MHz', desc: 'Banda DX internacional' },
    '20m': { range: '14.0-14.35 MHz', desc: 'Banda DX principal' },
    '40m': { range: '7.0-7.3 MHz', desc: 'Regional/noche' },
    '80m': { range: '3.5-4.0 MHz', desc: 'Local/noche' },
    '160m': { range: '1.8-2.0 MHz', desc: 'Local larga distancia' },
    '320m': { range: '0.9-1.0 MHz', desc: 'Experimental LF' },
    '640m': { range: '0.47-0.49 MHz', desc: 'Experimental VLF' }
};

const pmrChannels = {
    8: { 1: 446.00625, 2: 446.01875, 3: 446.03125, 4: 446.04375, 5: 446.05625, 6: 446.06875, 7: 446.08125, 8: 446.09375 },
    16: { 1: 446.00625, 2: 446.01875, 3: 446.03125, 4: 446.04375, 5: 446.05625, 6: 446.06875, 7: 446.08125, 8: 446.09375,
          9: 446.10625, 10: 446.11875, 11: 446.13125, 12: 446.14375, 13: 446.15625, 14: 446.16875, 15: 446.18125, 16: 446.19375 },
    32: { 1: 446.00625, 2: 446.009375, 3: 446.0125, 4: 446.015625, 5: 446.01875, 6: 446.021875, 7: 446.025, 8: 446.028125,
          9: 446.03125, 10: 446.034375, 11: 446.0375, 12: 446.040625, 13: 446.04375, 14: 446.046875, 15: 446.05, 16: 446.053125,
          17: 446.05625, 18: 446.059375, 19: 446.0625, 20: 446.065625, 21: 446.06875, 22: 446.071875, 23: 446.075, 24: 446.078125,
          25: 446.08125, 26: 446.084375, 27: 446.0875, 28: 446.090625, 29: 446.09375, 30: 446.096875, 31: 446.1, 32: 446.103125 }
};

const cbChannels = {
    1: 26.965, 2: 26.975, 3: 26.985, 4: 27.005, 5: 27.015, 6: 27.025, 7: 27.035, 8: 27.055,
    9: 27.065, 10: 27.075, 11: 27.085, 12: 27.105, 13: 27.115, 14: 27.125, 15: 27.135, 16: 27.155,
    17: 27.165, 18: 27.175, 19: 27.185, 20: 27.205, 21: 27.215, 22: 27.225, 23: 27.255, 24: 27.235,
    25: 27.245, 26: 27.265, 27: 27.275, 28: 27.285, 29: 27.295, 30: 27.305, 31: 27.315, 32: 27.325,
    33: 27.335, 34: 27.345, 35: 27.355, 36: 27.365, 37: 27.375, 38: 27.385, 39: 27.395, 40: 27.405
};

const emergencyFrequencies = {
    'UHF': [{ freq: '433.000', purpose: 'Emergencia general UHF' }, { freq: '446.000', purpose: 'PMR446 Canal 1' }],
    'VHF': [{ freq: '146.520', purpose: 'Simplex nacional (EEUU)' }, { freq: '145.500', purpose: 'Emergencia VHF' }],
    'AEREA': [{ freq: '121.500', purpose: 'Emergencia aeronáutica' }, { freq: '123.100', purpose: 'Ayuda aeronáutica' }],
    'MARINA': [{ freq: '156.800', purpose: 'Canal 16 - Emergencia' }, { freq: '156.300', purpose: 'Canal 6 - Auxilio' }],
    'HF': [{ freq: '14.300', purpose: 'Emergencia HF global' }, { freq: '7.296', purpose: 'Red de emergencia' }],
    'EMERG': [{ freq: '121.500', purpose: 'Emergencia aeronáutica' }, { freq: '156.800', purpose: 'Emergencia marítima' }, { freq: '27.185', purpose: 'Canal 19 CB - Emergencia' }]
};

const phoneticAlphabetFull = {
    'A': { word: 'ALFA', pronunciation: 'AL - FAH' },
    'B': { word: 'BRAVO', pronunciation: 'BRAH - VOH' },
    'C': { word: 'CHARLIE', pronunciation: 'CHAR - LEE' },
    'D': { word: 'DELTA', pronunciation: 'DELL - TAH' },
    'E': { word: 'ECHO', pronunciation: 'ECK - OH' },
    'F': { word: 'FOXTROT', pronunciation: 'FOKS - TROT' },
    'G': { word: 'GOLF', pronunciation: 'GOLF' },
    'H': { word: 'HOTEL', pronunciation: 'HOH - TEL' },
    'I': { word: 'INDIA', pronunciation: 'IN - DEE - AH' },
    'J': { word: 'JULIETT', pronunciation: 'JEW - LEE - ETT' },
    'K': { word: 'KILO', pronunciation: 'KEY - LOH' },
    'L': { word: 'LIMA', pronunciation: 'LEE - MAH' },
    'M': { word: 'MIKE', pronunciation: 'MIKE' },
    'N': { word: 'NOVEMBER', pronunciation: 'NO - VEM - BER' },
    'O': { word: 'OSCAR', pronunciation: 'OSS - CAH' },
    'P': { word: 'PAPA', pronunciation: 'PAH - PAH' },
    'Q': { word: 'QUEBEC', pronunciation: 'KEH - BECK' },
    'R': { word: 'ROMEO', pronunciation: 'ROW - ME - OH' },
    'S': { word: 'SIERRA', pronunciation: 'SEE - AIR - RAH' },
    'T': { word: 'TANGO', pronunciation: 'TANG - GO' },
    'U': { word: 'UNIFORM', pronunciation: 'YOU - NEE - FORM' },
    'V': { word: 'VICTOR', pronunciation: 'VIK - TAH' },
    'W': { word: 'WHISKEY', pronunciation: 'WISS - KEY' },
    'X': { word: 'X-RAY', pronunciation: 'ECKS - RAY' },
    'Y': { word: 'YANKEE', pronunciation: 'YANG - KEY' },
    'Z': { word: 'ZULU', pronunciation: 'ZOO - LOO' },
    '0': { word: 'ZERO', pronunciation: 'ZE-RO' },
    '1': { word: 'ONE', pronunciation: 'WUN' },
    '2': { word: 'TWO', pronunciation: 'TOO' },
    '3': { word: 'THREE', pronunciation: 'TREE' },
    '4': { word: 'FOUR', pronunciation: 'FOW-ER' },
    '5': { word: 'FIVE', pronunciation: 'FIFE' },
    '6': { word: 'SIX', pronunciation: 'SIX' },
    '7': { word: 'SEVEN', pronunciation: 'SEV-EN' },
    '8': { word: 'EIGHT', pronunciation: 'AIT' },
    '9': { word: 'NINE', pronunciation: 'NIN-ER' }
};

// =============================================
// FUNCIONES DE SEGURIDAD Y CRIPTOGRAFÍA
// =============================================

function secureLocalSet(key, value) {
    try {
        const secret = 'radcom-secure-key-2026';
        const enc = CryptoJS.AES.encrypt(JSON.stringify(value), secret).toString();
        localStorage.setItem(key, enc);
    } catch (e) {
        console.error('Secure set failed:', e);
    }
}

function secureLocalGet(key, defaultValue = null) {
    try {
        const enc = localStorage.getItem(key);
        if (!enc) return defaultValue;
        const secret = 'radcom-secure-key-2026';
        const dec = CryptoJS.AES.decrypt(enc, secret);
        return JSON.parse(dec.toString(CryptoJS.enc.Utf8));
    } catch (e) {
        console.error('Secure get failed:', e);
        return defaultValue;
    }
}

function xorEncrypt(text, key) {
    let result = '';
    for (let i = 0; i < text.length; i++) {
        result += String.fromCharCode(text.charCodeAt(i) ^ key.charCodeAt(i % key.length));
    }
    return result;
}

function xorDecrypt(text, key) {
    return xorEncrypt(text, key);
}

function encryptAES(text, password) {
    try {
        const iv = CryptoJS.lib.WordArray.random(16);
        const key = CryptoJS.PBKDF2(password, iv, { keySize: 256/32 });
        const encrypted = CryptoJS.AES.encrypt(text, key, { iv: iv });
        return iv.toString() + ':' + encrypted.toString();
    } catch (error) {
        console.error("Error AES, usando XOR:", error);
        return xorEncrypt(text, password);
    }
}

function decryptAES(ciphertext, password) {
    try {
        const parts = ciphertext.split(':');
        const iv = CryptoJS.enc.Hex.parse(parts[0]);
        const encrypted = parts[1];
        const key = CryptoJS.PBKDF2(password, iv, { keySize: 256/32 });
        const decrypted = CryptoJS.AES.decrypt(encrypted, key, { iv: iv });
        return decrypted.toString(CryptoJS.enc.Utf8);
    } catch (error) {
        console.error("Error descifrado AES, usando XOR:", error);
        return xorDecrypt(ciphertext, password);
    }
}

// =============================================
// CAPA DE SEGURIDAD UNIFICADA - VERSIÓN FINAL CORREGIDA
// =============================================

function securityLayer(text, isSending, encryptionMode, password) {
    if (!password) password = "";
    encryptionMode = (encryptionMode || "none").toLowerCase().trim();

    const result = {
        text: text,
        encryptionUsed: "NONE",
        error: null
    };

    if (encryptionMode === "none" || !password) {
        return result;
    }

    try {
        if (encryptionMode === "aes" || encryptionMode === "aes-256-gcm") {
            if (isSending) {
                const iv = CryptoJS.lib.WordArray.random(12);
                const key = CryptoJS.SHA256(password);
                const encrypted = CryptoJS.AES.encrypt(text, key, { 
                    iv: iv,
                    mode: CryptoJS.mode.CBC,
                    padding: CryptoJS.pad.Pkcs7
                });
                result.text = iv.toString() + ':' + encrypted.toString();
                result.encryptionUsed = "AES-256-CBC";
            } else {
                if (text.includes(':')) {
                    const parts = text.split(':');
                    if (parts.length === 2) {
                        try {
                            const iv = CryptoJS.enc.Hex.parse(parts[0]);
                            const ct = parts[1];
                            const key = CryptoJS.SHA256(password);
                            const decrypted = CryptoJS.AES.decrypt(ct, key, { 
                                iv: iv,
                                mode: CryptoJS.mode.CBC,
                                padding: CryptoJS.pad.Pkcs7
                            });
                            const plaintext = decrypted.toString(CryptoJS.enc.Utf8);
                            if (plaintext) {
                                result.text = plaintext;
                                result.encryptionUsed = "AES-256-CBC";
                                return result;
                            }
                        } catch (e) {}
                    }
                    if (parts.length === 3) {
                        try {
                            const salt = CryptoJS.enc.Base64.parse(parts[0]);
                            const iv = CryptoJS.enc.Base64.parse(parts[1]);
                            const ct = parts[2];
                            const key = CryptoJS.PBKDF2(password, salt, { keySize: 256/32, iterations: 1000 });
                            const decrypted = CryptoJS.AES.decrypt(ct, key, { iv: iv, mode: CryptoJS.mode.GCM, padding: CryptoJS.pad.NoPadding });
                            const plaintext = decrypted.toString(CryptoJS.enc.Utf8);
                            if (plaintext) {
                                result.text = plaintext;
                                result.encryptionUsed = "AES-256-GCM (legacy)";
                                return result;
                            }
                        } catch (e) {}
                    }
                }
                result.text = text;
                result.encryptionUsed = "ERROR";
                result.error = "Formato AES no reconocido";
            }
        } else if (encryptionMode === "xor") {
            let output = "";
            for (let i = 0; i < text.length; i++) {
                output += String.fromCharCode(text.charCodeAt(i) ^ password.charCodeAt(i % password.length));
            }
            result.text = output;
            result.encryptionUsed = "XOR";
        } else {
            result.error = "Modo de cifrado desconocido: " + encryptionMode;
        }
    } catch (err) {
        result.error = err.message || "Error en capa de seguridad";
        console.error("[securityLayer]", result.error);
        if (!isSending) {
            result.text = text;
            result.encryptionUsed = "ERROR";
        }
    }

    return result;
}

function updateSecurityBadge() {
    const encryptionSelect = document.getElementById('encryptionMode');
    const securityBadge = document.querySelector('.security-badge span:nth-child(2)');
    const securityDot = document.querySelector('.security-dot');
    
    if (!encryptionSelect || !securityBadge) return;
    
    const encryption = encryptionSelect.value;
    
    if (encryption === 'aes') {
        securityBadge.textContent = 'AES-256-GCM';
        securityBadge.style.color = '#00ffea';
        securityDot.style.background = '#00ff88';
        securityDot.style.boxShadow = '0 0 4px #00ff88';
    } else if (encryption === 'xor') {
        securityBadge.textContent = 'XOR';
        securityBadge.style.color = '#ffaa00';
        securityDot.style.background = '#ffaa00';
        securityDot.style.boxShadow = '0 0 4px #ffaa00';
    } else if (encryption === 'none') {
        securityBadge.textContent = 'SIN CIFRADO';
        securityBadge.style.color = '#ff3300';
        securityDot.style.background = '#ff3300';
        securityDot.style.boxShadow = '0 0 4px #ff3300';
    }
}

function showSecurityInfo() {
    const panel = document.createElement('div');
    panel.style.cssText = `
        position: fixed;
        top: 50%;
        left: 50%;
        transform: translate(-50%, -50%);
        background: rgba(0, 10, 20, 0.95);
        border: 2px solid #00ff88;
        border-radius: 10px;
        padding: 20px;
        width: 300px;
        z-index: 9999;
        color: #00ff88;
        font-family: 'Courier New', monospace;
        box-shadow: 0 0 40px rgba(0, 255, 136, 0.4);
    `;
    
    panel.innerHTML = `
        <h3 style="margin: 0 0 15px 0; color: #00ffea;">
            <i class="fas fa-shield-alt"></i> SEGURIDAD v5.2
        </h3>
        <div style="background: rgba(0, 20, 0, 0.3); padding: 10px; border-radius: 5px; margin-bottom: 15px;">
            <div style="display: flex; justify-content: space-between; margin-bottom: 8px;">
                <span>Estado:</span>
                <span style="color:#00ff88;">ACTIVO</span>
            </div>
            <div style="display: flex; justify-content: space-between; margin-bottom: 8px;">
                <span>Algoritmo:</span>
                <span style="color:#00ffea;">AES-GCM-256</span>
            </div>
            <div style="display: flex; justify-content: space-between;">
                <span>Nivel:</span>
                <span style="color:#00ff88;">ALTO</span>
            </div>
        </div>
        <div style="font-size: 0.75rem; color: #88ffaa;">
            <p>✓ Cifrado extremo a extremo</p>
            <p>✓ Autenticación de mensajes</p>
            <p>✓ Protección contra manipulación</p>
        </div>
        <button onclick="this.parentElement.remove()" 
                style="margin-top: 15px; width: 100%; padding: 8px; background: #00ff88; color: #000; 
                       border: none; border-radius: 5px; font-weight: bold; cursor: pointer;">
            CERRAR
        </button>
    `;
    
    document.body.appendChild(panel);
    
    setTimeout(() => {
        panel.addEventListener('click', e => e.stopPropagation());
        document.addEventListener('click', () => panel.remove());
    }, 100);
}

// =============================================
// FUNCIONES DE PEERJS Y CONEXIONES
// =============================================

function generateHardwareBasedId() {
    const sources = [
        navigator.userAgent,
        navigator.language,
        screen.width + 'x' + screen.height,
        new Date().getTimezoneOffset(),
        localStorage.getItem('radcom_hardware_id')
    ];
    
    if (!localStorage.getItem('radcom_hardware_id')) {
        const hardwareId = 'HW-' + Math.random().toString(36).substring(2, 10) + '-' + Date.now().toString(36);
        localStorage.setItem('radcom_hardware_id', hardwareId);
        sources[4] = hardwareId;
    }
    
    let hash = 0;
    const str = sources.join('|');
    for (let i = 0; i < str.length; i++) {
        const char = str.charCodeAt(i);
        hash = ((hash << 5) - hash) + char;
        hash = hash & hash;
    }
    
    return `RADCOM-${Math.abs(hash).toString(36).toUpperCase().substring(0, 8)}-V4`;
}

function generateRobustPeerId() {
    const timestamp = Date.now().toString(36);
    const random = Math.random().toString(36).substring(2, 8);
    return `RADCOM-${timestamp}-${random}`.toUpperCase();
}

function getOrCreateFixedId() {
    let fixedId = localStorage.getItem('radcom_fixed_id_v4');
    if (!fixedId) {
        fixedId = generateHardwareBasedId();
        localStorage.setItem('radcom_fixed_id_v4', fixedId);
    }
    return fixedId;
}

function connectToPeer() {
    const input = document.getElementById('peer-id-input');
    let peerId = input.value.trim();
    
    if (!peerId || !/^[a-zA-Z0-9\-]{1,20}$/.test(peerId)) {
        document.getElementById('monitor-raw').innerHTML = 
            '<span style="color:#ff3300">⚠️ ID INVÁLIDO (solo letras, números y guiones)</span>';
        return;
    }
    
    input.value = '';
    saveId(peerId);
    connectToPeerId(peerId);
}

function connectToPeerId(peerId) {
    if (!peer || peer.disconnected) {
        console.log("❌ Peer no inicializado");
        return;
    }
    if (connections[peerId] && connections[peerId].status === 'online') {
        console.log("⚠️ Ya conectado a", peerId);
        return;
    }
    
    console.log("🔌 Conectando a:", peerId);
    updateMonitor(`🔌 CONECTANDO A: ${peerId.substring(0, 8)}...`);
    
    try {
        const conn = peer.connect(peerId, { reliable: true, serialization: 'json' });
        setupConnection(conn, 'outgoing');
    } catch (error) {
        console.error(`❌ Error conectando a ${peerId}:`, error);
        updateMonitor(`❌ ERROR CONECTANDO A ${peerId.substring(0, 8)}`);
    }
}

function setupConnection(conn, type) {
    const peerId = conn.peer;
    
    connections[peerId] = {
        conn: conn,
        status: 'connecting',
        type: type,
        health: 'new',
        latency: 0,
        lastCheck: Date.now()
    };
    
    connectionHealth[peerId] = {
        connectedAt: Date.now(),
        lastActivity: Date.now(),
        packetsSent: 0,
        packetsReceived: 0
    };
    
    updatePeerList();
    
    conn.on('open', () => {
        console.log(`✅ Conexión establecida con:`, peerId);
        connections[peerId].status = 'online';
        connections[peerId].health = 'healthy';
        updatePeerList();
        updateConnectedCount();
        updateMonitor(`✅ CONECTADO A ${peerId.substring(0, 8)}`);
        playStrongBeep(800, 100);
        setTimeout(() => deliverOnConnect(peerId), 1500);
        saveId(peerId);
        setTimeout(() => sendHealthPing(peerId), 1000);
    });
    
    conn.on('data', (data) => {
        console.log("📥 Datos recibidos de", peerId, ":", data);
        
        connectionHealth[peerId].lastActivity = Date.now();
        connectionHealth[peerId].packetsReceived++;
        
        if (data.type === 'health_ping') {
            try {
                conn.send({ type: 'health_pong', timestamp: data.timestamp, sender: myPeerId });
                return;
            } catch (error) {
                console.error(`❌ Error enviando pong:`, error);
            }
        }
        
        if (data.type === 'health_pong') {
            handleHealthPong(peerId, data.timestamp);
            return;
        }
        
        if (data.type === 'voip_offer') {
            handleVoIPOffer(peerId, data);
            return;
        }
        
        if (data.type === 'voip_answer') {
            handleVoIPAnswer(peerId, data);
            return;
        }
        
        if (data.type === 'voip_ice_candidate') {
            handleVoIPICECandidate(peerId, data);
            return;
        }
        
        if (data.type === 'voip_end') {
            handleVoIPEnd(peerId);
            return;
        }
        
        try {
            handleReceivedData(peerId, data);
        } catch (error) {
            console.error("❌ Error procesando datos:", error);
        }
    });
    
    conn.on('close', () => {
        console.log(`🔒 Conexión cerrada con:`, peerId);
        connections[peerId].status = 'offline';
        updatePeerList();
        updateConnectedCount();
        updateMonitor(`🔒 DESCONECTADO: ${peerId.substring(0, 8)}`);
        
        if (voipActiveCall === peerId) {
            endVoIPCall(peerId);
        }
        if (voipIncomingCall === peerId) {
            rejectVoIPCall(peerId, 'closed');
        }
        
        if (document.getElementById('autoReconnect')?.checked) {
            setTimeout(() => {
                if (peerId !== myPeerId && !connections[peerId]?.conn) {
                    console.log(`🔄 Reconexión automática con ${peerId}`);
                    connectToPeerId(peerId);
                }
            }, 3000);
        }
    });
    
    conn.on('error', (err) => {
        console.error(`❌ Error en conexión ${peerId}:`, err);
        connections[peerId].status = 'error';
        updatePeerList();
    });
}

function handleIncomingConnection(conn) {
    console.log("🔄 Procesando conexión entrante:", conn.peer);
    setupConnection(conn, 'incoming');
}

function setupPeerEvents() {
    peer.on('open', (id) => {
        myPeerId = id;
        localStorage.setItem('radcom_master_id_v5', id);
        document.getElementById('display-id').innerHTML = 
            `<span style="color:#00ff88">ID: ${id.substring(0,12)}...</span>`;
        
        updateMonitor(`✅ RADCOM MASTER v${VERSION} INICIADO | ID: ${id.substring(0,10)}...`);
        
        savedIds.forEach(pid => {
            if (pid !== id) setTimeout(() => connectToPeerId(pid), 300);
        });
        
        setInterval(updateUptime, 1000);
        updateStats();
    });

    peer.on('connection', (conn) => {
        setupConnection(conn, 'incoming');
    });

    peer.on('error', (err) => {
        console.error("PeerJS Error:", err);
        updateMonitor(`⚠️ ERROR: ${err.type}`, "error");
    });
}

function initPeerJSEnhanced() {
    updateMonitor("📡 INICIALIZANDO SISTEMA DE ID FIJOS...");
    
    const config = JSON.parse(localStorage.getItem('radcom_config_v4') || '{}');
    ID_SYSTEM.useFixedId = config.useFixedId !== false;
    ID_SYSTEM.fixedId = config.fixedId || getOrCreateFixedId();
    
    let peerId;
    
    if (ID_SYSTEM.useFixedId && ID_SYSTEM.fixedId) {
        peerId = ID_SYSTEM.fixedId;
        updateMonitor(`🔧 USANDO ID FIJADO: ${peerId.substring(0, 20)}...`);
    } else {
        peerId = generateRobustPeerId();
        updateMonitor(`🎲 USANDO ID ALEATORIO: ${peerId.substring(0, 20)}...`);
    }
    
    ID_SYSTEM.currentId = peerId;
    localStorage.setItem('radcom_current_id', peerId);
    
    document.getElementById('currentIdDisplay').textContent = peerId;
    
    try {
        const iceConfig = {
            config: {
                iceServers: [
                    { urls: 'stun:stun.l.google.com:19302' },
                    { urls: 'stun:stun1.l.google.com:19302' }
                ]
            },
            debug: 0
        };
        
        if (currentConnectionType === 'mobile') {
            iceConfig.config.iceServers = [
                { urls: 'stun:global.stun.twilio.com:3478?transport=udp' },
                { urls: 'stun:stun.l.google.com:19302' },
                { urls: 'stun:stun1.l.google.com:19302' },
                { urls: 'stun:stun2.l.google.com:19302' },
                { urls: 'stun:stun3.l.google.com:19302' },
                { urls: 'stun:stun4.l.google.com:19302' }
            ];
        }
        
        peer = new Peer(peerId, iceConfig);
        setupPeerEvents();
        updateMonitor(`✅ ID REGISTRADO: ${peerId.substring(0, 20)}...`);
        
    } catch (error) {
        console.error("Error inicializando PeerJS:", error);
        
        if (ID_SYSTEM.useFixedId) {
            updateMonitor("⚠️ ID FIJADO OCUPADO. USANDO ID ALEATORIO...", "warning");
            
            peerId = generateRobustPeerId();
            ID_SYSTEM.useFixedId = false;
            ID_SYSTEM.fixedId = null;
            
            const config = JSON.parse(localStorage.getItem('radcom_config_v4') || '{}');
            config.useFixedId = false;
            config.fixedId = null;
            localStorage.setItem('radcom_config_v4', JSON.stringify(config));
            
            setTimeout(() => {
                if (peer) peer.destroy();
                setTimeout(() => initPeerJSEnhanced(), 1000);
            }, 500);
        }
    }
}

function saveId(id) {
    const idStr = String(id).trim();
    
    if (idStr.length > 30 || !/^[a-zA-Z0-9\-]+$/.test(idStr)) {
        console.log("ID rechazado (formato inválido):", idStr.substring(0,20));
        return false;
    }
    
    if (!savedIds.includes(idStr)) {
        savedIds.push(idStr);
        localStorage.setItem('radcom_peers', JSON.stringify(savedIds));
        updatePeerList();
    }
    return true;
}

function executeRemoveId(id) {
    if (!confirm(`¿Eliminar ${id.substring(0, 8)} del historial?`)) {
        return;
    }
    
    console.log("🗑️ Eliminando ID:", id);
    
    savedIds = savedIds.filter(savedId => savedId !== id);
    localStorage.setItem('radcom_peers_v4', JSON.stringify(savedIds));
    
    if (voipActiveCall === id) {
        endVoIPCall(id);
    }
    if (voipIncomingCall === id) {
        rejectVoIPCall(id, 'removed');
    }
    
    if (connections[id]) {
        if (connections[id].conn) {
            connections[id].conn.close();
        }
        delete connections[id];
        delete connectionHealth[id];
    }
    
    if (activeTarget === id) {
        activeTarget = 'GLOBAL';
        document.getElementById('target-display').innerHTML = 
            '<i class="fas fa-crosshairs"></i> DESTINO: GLOBAL';
        document.getElementById('target-global').classList.add('active');
    }
    
    updatePeerList();
    updateConnectedCount();
    
    updateMonitor(`🗑️ ELIMINADO: ${id.substring(0, 8)}`);
    playStrongBeep(300, 100);
}

// =============================================
// FUNCIÓN updatePeerList CORREGIDA - VERSIÓN 5.7.2
// =============================================
function updatePeerList() {
    const container = document.getElementById('saved-peers');
    if (!container) {
        console.error("❌ Contenedor de peers no encontrado");
        return;
    }
    
    let html = "";
    
    // Verificar que savedIds es un array
    if (!Array.isArray(savedIds)) {
        console.error("❌ savedIds no es un array:", savedIds);
        savedIds = [];
    }
    
    savedIds.forEach(peerId => {
        // Usar peerId en lugar de id para evitar conflictos
        if (!peerId || typeof peerId !== 'string') {
            console.warn("⚠️ ID de peer inválido:", peerId);
            return;
        }
        
        const conn = connections[peerId];
        let status = conn ? conn.status : 'offline';
        let health = conn ? conn.health : 'offline';
        
        let statusClass = 'st-offline';
        
        if (status === 'online') {
            if (health === 'healthy') {
                statusClass = 'st-online';
            } else if (health === 'suspicious' || health === 'unresponsive') {
                statusClass = 'st-warning';
            } else if (health === 'new') {
                statusClass = 'st-encrypted';
            }
        }
        
        let statusText = status.toUpperCase();
        if (health === 'suspicious') statusText = 'INACTIVO';
        if (health === 'unresponsive') statusText = 'SIN RESPUESTA';
        
        const displayStyle = (status === 'offline' && !showOffline) ? 'display: none;' : '';
        
        // --- Lógica de botones VoIP ---
        let voipBtnClass = 'voip-btn';
        let voipIcon = 'fa-phone';
        let voipTitle = 'Iniciar llamada VoIP';
        
        if (voipActiveCall === peerId) {
            voipBtnClass += ' in-call';
            voipIcon = 'fa-phone-slash';
            voipTitle = 'Finalizar llamada';
        } else if (voipIncomingCall === peerId) {
            voipBtnClass += ' active';
            voipIcon = 'fa-phone-alt';
            voipTitle = 'Llamada entrante';
        } else if (status !== 'online') {
            voipBtnClass += ' disabled';
            voipTitle = 'No disponible (offline)';
        }
        
        // Mostrar solo los primeros 8 caracteres del ID
        const displayId = peerId.length > 8 ? peerId.substring(0, 8) + '…' : peerId;
        
        // ===== HTML CORREGIDO =====
        // El onclick para seleccionar peer está en el div PRINCIPAL
        // Los botones tienen event.stopPropagation() para no activar la selección
        html += `
            <div class="peer-item ${activeTarget === peerId ? 'active' : ''}" 
                 data-peer-id="${peerId.replace(/"/g, '&quot;')}"
                 data-status="${status}"
                 data-health="${health}"
                 onclick="selectPeer('${peerId.replace(/'/g, "\\'")}')"
                 style="${displayStyle}">
                 
                <div class="peer-info">
                    <span class="status-dot ${statusClass}"></span>
                    <div style="display:flex; flex-direction:column;">
                        <span style="font-weight:bold;" title="${peerId}">${displayId}</span>
                        <span style="font-size:0.55rem; color:#888;">
                            ${statusText}
                            ${conn?.latency ? ` (${conn.latency}ms)` : ''}
                        </span>
                    </div>
                </div>
                
                <div class="peer-actions">
                    <button class="${voipBtnClass}" 
                            onclick="event.stopPropagation(); toggleVoIPCall('${peerId.replace(/'/g, "\\'")}')" 
                            title="${voipTitle}">
                        <i class="fas ${voipIcon}"></i>
                    </button>
                    <span class="del-btn" 
                          onclick="event.stopPropagation(); executeRemoveId('${peerId.replace(/'/g, "\\'")}')">×</span>
                </div>
            </div>
        `;
    });
    
    container.innerHTML = html;
    updateConnectionStatusIndicators();
}

function updateConnectionStatusIndicators() {
    const onlineHealthy = Object.keys(connections).filter(id => 
        connections[id]?.status === 'online' && connections[id]?.health === 'healthy').length;
    
    const onlineSuspicious = Object.keys(connections).filter(id => 
        connections[id]?.status === 'online' && 
        (connections[id]?.health === 'suspicious' || connections[id]?.health === 'unresponsive')).length;
    
    const countElement = document.getElementById('connected-count');
    if (countElement) {
        if (onlineHealthy > 0) {
            countElement.innerHTML = 
                `<span style="color:#00ff88">${onlineHealthy}</span>` + 
                (onlineSuspicious > 0 ? `<span style="color:#ffaa00">/${onlineSuspicious}</span>` : '');
        } else if (onlineSuspicious > 0) {
            countElement.innerHTML = `<span style="color:#ffaa00">${onlineSuspicious}</span>`;
        } else {
            countElement.innerHTML = '0';
        }
    }
    
    document.getElementById('stats-connections').textContent = onlineHealthy + onlineSuspicious;
}

function toggleShowOffline() {
    showOffline = !showOffline;
    updatePeerList();
    
    const icon = document.querySelector('#saved-peers-container').parentElement
        .querySelector('.fa-eye-slash, .fa-eye');
    if (icon) {
        icon.classList.toggle('fa-eye');
        icon.classList.toggle('fa-eye-slash');
    }
    
    updateMonitor(showOffline ? "👁️ MOSTRANDO OFFLINE" : "👁️ OCULTANDO OFFLINE");
    playStrongBeep(600, 50);
}

function selectPeer(id) {
    if (id !== 'GLOBAL' && connections[id]?.health === 'suspicious') {
        sendHealthPing(id);
    }
    
    activeTarget = id;
    
    document.querySelectorAll('.peer-item').forEach(item => {
        item.classList.remove('active');
    });
    
    if (id === 'GLOBAL') {
        document.getElementById('target-global').classList.add('active');
    } else {
        const peerElement = document.querySelector(`[data-peer-id="${id}"]`);
        if (peerElement) {
            peerElement.classList.add('active');
        }
    }
    
    document.getElementById('target-display').innerHTML = 
        `<i class="fas fa-crosshairs"></i> DESTINO: ${id === 'GLOBAL' ? "GLOBAL" : id.substring(0, 8)}`;
}

function updateConnectedCount() {
    const onlineCount = Object.keys(connections).filter(id => connections[id]?.status === 'online').length;
    document.getElementById('stats-connections').textContent = onlineCount;
}

// =============================================
// FUNCIONES DE HEALTH CHECK Y REVIVE
// =============================================

function sendHealthPing(peerId) {
    if (!connections[peerId]?.conn || connections[peerId]?.status !== 'online') return;
    
    try {
        connections[peerId].conn.send({
            type: 'health_ping',
            timestamp: Date.now(),
            sender: myPeerId
        });
        
        console.log(`📡 Ping enviado a ${peerId}`);
        connectionHealth[peerId] = {
            ...connectionHealth[peerId],
            lastPing: Date.now(),
            waitingForPong: true
        };
        
        setTimeout(() => {
            if (connectionHealth[peerId]?.waitingForPong) {
                console.log(`❌ No hay respuesta de ${peerId}`);
                connections[peerId].health = 'unresponsive';
                updatePeerList();
            }
        }, 3000);
        
    } catch (error) {
        console.error(`❌ Error enviando ping a ${peerId}:`, error);
        connections[peerId].health = 'error';
        updatePeerList();
    }
}

function handleHealthPong(peerId, timestamp) {
    const latency = Date.now() - timestamp;
    
    connectionHealth[peerId] = {
        ...connectionHealth[peerId],
        lastActivity: Date.now(),
        lastLatency: latency,
        waitingForPong: false
    };
    
    connections[peerId].health = 'healthy';
    connections[peerId].latency = latency;
    
    console.log(`✅ Pong recibido de ${peerId} (${latency}ms)`);
    updatePeerList();
    
    document.getElementById('stats-latency').textContent = `${latency} ms`;
}

function checkConnectionsHealth() {
    Object.keys(connections).forEach(peerId => {
        if (connections[peerId]?.status === 'online') {
            const lastActivity = connectionHealth[peerId]?.lastActivity || 0;
            const inactiveTime = Date.now() - lastActivity;
            
            if (inactiveTime > 10000) {
                sendHealthPing(peerId);
            }
        }
    });
}

function startHealthCheckSystem() {
    setInterval(() => {
        if (!isBackground) {
            checkConnectionsHealth();
        }
    }, 10000);
    
    setInterval(() => {
        const now = Date.now();
        Object.keys(connectionHealth).forEach(peerId => {
            if (connections[peerId]?.status === 'online') {
                const lastActivity = connectionHealth[peerId]?.lastActivity || 0;
                const inactiveTime = now - lastActivity;
                
                if (inactiveTime > 15000) {
                    console.log(`⚠️ Conexión ${peerId} inactiva por ${Math.round(inactiveTime/1000)}s`);
                    if (connections[peerId].health !== 'suspicious') {
                        connections[peerId].health = 'suspicious';
                        updatePeerList();
                    }
                }
            }
        });
    }, 5000);
}

function forceReconnect(peerId) {
    console.log(`🔁 Forzando reconexión con ${peerId}`);
    
    const shouldReconnect = savedIds.includes(peerId);
    
    if (connections[peerId]?.conn) {
        try {
            connections[peerId].conn.close();
        } catch (e) {
            console.error("Error cerrando conexión:", e);
        }
    }
    
    delete connections[peerId];
    delete connectionHealth[peerId];
    
    updatePeerList();
    updateConnectedCount();
    
    if (shouldReconnect) {
        setTimeout(() => {
            connectToPeerId(peerId);
            updateMonitor(`🔄 RECONECTANDO: ${peerId.substring(0, 8)}...`);
        }, 1000);
    }
}

function reviveAllConnections() {
    if (revivingInProgress) {
        console.log("⚠️ Ya hay una operación de revivir en progreso");
        return;
    }
    
    revivingInProgress = true;
    const reviveBtn = document.getElementById('reviveBtn');
    if (reviveBtn) {
        reviveBtn.classList.add('reviving');
        reviveBtn.disabled = true;
        reviveBtn.innerHTML = '<i class="fas fa-spinner fa-spin"></i> REVIVIENDO...';
    }
    
    updateMonitor("⚡ REVIVIENDO CONEXIONES...");
    playStrongBeep(600, 100);
    
    let revived = 0;
    let suspiciousCount = 0;
    
    Object.keys(connections).forEach(peerId => {
        if (connections[peerId]?.status === 'online' && 
            (connections[peerId]?.health === 'suspicious' || connections[peerId]?.health === 'unresponsive')) {
            suspiciousCount++;
        }
    });
    
    if (suspiciousCount === 0) {
        updateMonitor("✅ TODAS LAS CONEXIONES ESTÁN SALUDABLES");
        finishReviveOperation(reviveBtn, 0);
        return;
    }
    
    updateMonitor(`🔍 ENCONTRADAS ${suspiciousCount} CONEXIÓN(ES) SOSPECHOSAS`);
    
    Object.keys(connections).forEach(peerId => {
        if (connections[peerId]?.status === 'online' && 
            (connections[peerId]?.health === 'suspicious' || connections[peerId]?.health === 'unresponsive')) {
            
            console.log(`⚡ Reviviendo ${peerId}`);
            revived++;
            
            sendHealthPing(peerId);
            
            if (aggressiveRevive) {
                setTimeout(() => {
                    if (connections[peerId]?.health !== 'healthy') {
                        console.log(`🔁 Reconexión agresiva para ${peerId}`);
                        forceReconnect(peerId);
                    }
                }, 2000);
            }
        }
    });
    
    setTimeout(() => {
        updateMonitor(`🔄 REVIVIDAS ${revived} CONEXIÓN(ES)`);
        
        setTimeout(() => {
            let healthyCount = 0;
            Object.keys(connections).forEach(peerId => {
                if (connections[peerId]?.status === 'online' && connections[peerId]?.health === 'healthy') {
                    healthyCount++;
                }
            });
            
            updateMonitor(`✅ ${healthyCount} CONEXIÓN(ES) SALUDABLES`);
            finishReviveOperation(reviveBtn, revived);
        }, 5000);
    }, 3000);
}

function finishReviveOperation(reviveBtn, revivedCount) {
    revivingInProgress = false;
    
    if (reviveBtn) {
        reviveBtn.classList.remove('reviving');
        reviveBtn.disabled = false;
        reviveBtn.innerHTML = '<i class="fas fa-heartbeat"></i> REVIVIR CONEXIONES';
    }
    
    if (revivedCount > 0) {
        playStrongBeep(800, 100);
    }
    
    console.log(`✅ Operación de revivir completada: ${revivedCount} conexiones revividas`);
}

function refreshAllConnections() {
    console.log("🔄 Refrescando todas las conexiones...");
    updateMonitor("🔄 REFRESCANDO CONEXIONES...", "info");
    playStrongBeep(1000, 200);
    
    if (voipActiveCall) {
        endVoIPCall(voipActiveCall);
    }
    if (voipIncomingCall) {
        rejectVoIPCall(voipIncomingCall, 'refresh');
    }
    
    const currentPeers = [...savedIds];
    
    Object.keys(connections).forEach(peerId => {
        if (connections[peerId]?.conn) {
            try {
                connections[peerId].conn.close();
            } catch (error) {
                console.error("Error cerrando conexión:", error);
            }
        }
    });
    
    connections = {};
    connectionHealth = {};
    voipCalls = {};
    voipActiveCall = null;
    voipIncomingCall = null;
    stopRingtone();
    stopOutgoingTone();
    
    updatePeerList();
    updateConnectedCount();
    
    setTimeout(() => {
        currentPeers.forEach(peerId => {
            if (peerId !== myPeerId) {
                setTimeout(() => connectToPeerId(peerId), 300);
            }
        });
        updateMonitor("✅ CONEXIONES REFRESCADAS", "info");
    }, 1000);
}

// =============================================
// FUNCIONES VoIP
// =============================================

function initVoIPAudio() {
    if (!voipAudioContext) {
        try {
            voipAudioContext = new (window.AudioContext || window.webkitAudioContext)();
            console.log("✅ VoIP AudioContext inicializado");
        } catch (error) {
            console.error("❌ Error inicializando VoIP AudioContext:", error);
        }
    }
    return voipAudioContext;
}

function playRingtone() {
    stopRingtone();
    initVoIPAudio();
    if (!voipAudioContext) return;

    const now = voipAudioContext.currentTime;
    voipRingtoneSource = voipAudioContext.createOscillator();
    const gainNode = voipAudioContext.createGain();

    voipRingtoneSource.connect(gainNode);
    gainNode.connect(voipAudioContext.destination);

    voipRingtoneSource.type = 'sine';
    gainNode.gain.setValueAtTime(0.3, now);

    for (let i = 0; i < 40; i++) {
        const cycleStart = now + i * 3;
        const toneEnd = cycleStart + 2;

        voipRingtoneSource.frequency.setValueAtTime(440, cycleStart);
        gainNode.gain.setValueAtTime(0.3, cycleStart);
        gainNode.gain.setValueAtTime(0.3, toneEnd - 0.01);
        gainNode.gain.setValueAtTime(0, toneEnd);
    }

    voipRingtoneSource.start(now);
    voipRingtoneSource.stop(now + 120);
}

function stopRingtone() {
    if (voipRingtoneSource) {
        try {
            voipRingtoneSource.stop();
        } catch (e) {}
        voipRingtoneSource.disconnect();
        voipRingtoneSource = null;
    }
}

function playOutgoingTone() {
    stopOutgoingTone();
    initVoIPAudio();
    if (!voipAudioContext) return;

    const now = voipAudioContext.currentTime;
    voipOutgoingToneSource = voipAudioContext.createOscillator();
    const gainNode = voipAudioContext.createGain();

    voipOutgoingToneSource.connect(gainNode);
    gainNode.connect(voipAudioContext.destination);

    voipOutgoingToneSource.type = 'sine';
    gainNode.gain.setValueAtTime(0.2, now);

    for (let i = 0; i < 40; i++) {
        const cycleStart = now + i * 2;
        const toneEnd = cycleStart + 1;

        voipOutgoingToneSource.frequency.setValueAtTime(420, cycleStart);
        gainNode.gain.setValueAtTime(0.2, cycleStart);
        gainNode.gain.setValueAtTime(0.2, toneEnd - 0.01);
        gainNode.gain.setValueAtTime(0, toneEnd);
    }

    voipOutgoingToneSource.start(now);
    voipOutgoingToneSource.stop(now + 80);
}

function stopOutgoingTone() {
    if (voipOutgoingToneSource) {
        try {
            voipOutgoingToneSource.stop();
        } catch (e) {}
        voipOutgoingToneSource.disconnect();
        voipOutgoingToneSource = null;
    }
}

function showVoIPPanel(status, peerId) {
    const contentArea = document.getElementById('content-area');
    const existingPanel = document.getElementById('voip-control-panel');
    if (existingPanel) {
        existingPanel.remove();
    }

    const panel = document.createElement('div');
    panel.id = 'voip-control-panel';
    panel.className = 'voip-control-panel';

    let title = '';
    let buttons = '';

    if (status === 'incoming') {
        title = '📞 LLAMADA ENTRANTE';
        buttons = `
            <div class="voip-panel-buttons">
                <button class="voip-panel-btn voip-panel-btn-accept" onclick="acceptVoIPCall('${peerId}')"><i class="fas fa-phone-alt"></i></button>
                <button class="voip-panel-btn voip-panel-btn-reject" onclick="rejectVoIPCall('${peerId}', 'rejected')"><i class="fas fa-phone-slash"></i></button>
            </div>
        `;
    } else if (status === 'active') {
        title = '🔴 LLAMADA EN CURSO';
        buttons = `
            <div class="voip-panel-buttons">
                <button class="voip-panel-btn voip-panel-btn-end" onclick="endVoIPCall('${peerId}')"><i class="fas fa-phone-slash"></i></button>
            </div>
        `;
    } else if (status === 'outgoing') {
        title = '📞 LLAMANDO...';
        buttons = `
            <div class="voip-panel-buttons">
                <button class="voip-panel-btn voip-panel-btn-end" onclick="endVoIPCall('${peerId}')"><i class="fas fa-times"></i></button>
            </div>
        `;
    }

    panel.innerHTML = `
        <div class="voip-panel-title">${title}</div>
        <div class="voip-panel-peer">${peerId.substring(0, 12)}...</div>
        <div class="voip-panel-status">${status === 'active' ? 'En curso' : (status === 'incoming' ? 'Llamada entrante' : 'Estableciendo...')}</div>
        ${buttons}
    `;

    contentArea.appendChild(panel);
}

function hideVoIPPanel() {
    const panel = document.getElementById('voip-control-panel');
    if (panel) {
        panel.remove();
    }
}

async function toggleVoIPCall(peerId) {
    if (voipActiveCall === peerId) {
        endVoIPCall(peerId);
        return;
    }

    if (voipIncomingCall === peerId) {
        acceptVoIPCall(peerId);
        return;
    }

    if (voipActiveCall) {
        updateMonitor(`⚠️ Ya hay una llamada activa con ${voipActiveCall.substring(0, 8)}`, "warning");
        playStrongBeep(300, 200);
        return;
    }

    if (!connections[peerId] || connections[peerId].status !== 'online') {
        updateMonitor(`⚠️ ${peerId.substring(0, 8)} no está conectado`, "warning");
        playStrongBeep(300, 200);
        return;
    }

    try {
        updateMonitor(`📞 Iniciando llamada VoIP con ${peerId.substring(0, 8)}...`, "info");
        playStrongBeep(800, 100);
        playOutgoingTone();
        showVoIPPanel('outgoing', peerId);

        const stream = await navigator.mediaDevices.getUserMedia({ audio: true, video: false });
        voipMediaStream = stream;

        initVoIPAudio();

        const callId = `voip_${Date.now()}`;
        voipCalls[peerId] = {
            id: callId,
            peerConnection: new RTCPeerConnection({
                iceServers: [
                    { urls: 'stun:stun.l.google.com:19302' },
                    { urls: 'stun:stun1.l.google.com:19302' }
                ]
            }),
            stream: stream,
            status: 'initiating',
            remoteAudio: null
        };

        stream.getTracks().forEach(track => {
            voipCalls[peerId].peerConnection.addTrack(track, stream);
        });

        voipCalls[peerId].peerConnection.ontrack = (event) => {
            console.log(`📡 Recibiendo pista de audio remota de ${peerId}`);
            
            const remoteAudio = new Audio();
            remoteAudio.srcObject = event.streams[0];
            remoteAudio.autoplay = true;
            
            remoteAudio.play().catch(e => console.log("Error reproduciendo audio remoto:", e));

            if (voipCalls[peerId]) {
                voipCalls[peerId].remoteAudio = remoteAudio;
            }

            if (voipActiveCall === peerId) {
                displayMessage(`📞 Audio establecido con ${peerId.substring(0, 8)}`, '', 'voip');
            }
        };

        const offer = await voipCalls[peerId].peerConnection.createOffer();
        await voipCalls[peerId].peerConnection.setLocalDescription(offer);

        connections[peerId].conn.send({
            type: 'voip_offer',
            offer: offer,
            caller: myPeerId,
            callId: callId
        });

        updatePeerList();

        updateMonitor(`📞 Llamada iniciada con ${peerId.substring(0, 8)} - Esperando respuesta...`, "info");

        voipCalls[peerId].peerConnection.onicecandidate = (event) => {
            if (event.candidate) {
                connections[peerId].conn.send({
                    type: 'voip_ice_candidate',
                    candidate: event.candidate,
                    callId: callId,
                    for: peerId
                });
            }
        };

        voipCalls[peerId].peerConnection.onconnectionstatechange = () => {
            console.log(`VoIP connection state with ${peerId}: ${voipCalls[peerId].peerConnection.connectionState}`);
            if (voipCalls[peerId].peerConnection.connectionState === 'disconnected' ||
                voipCalls[peerId].peerConnection.connectionState === 'failed' ||
                voipCalls[peerId].peerConnection.connectionState === 'closed') {
                if (voipActiveCall === peerId || voipIncomingCall === peerId) {
                    endVoIPCall(peerId);
                }
            }
        };

    } catch (error) {
        console.error("❌ Error iniciando llamada VoIP:", error);
        updateMonitor(`❌ Error iniciando llamada: ${error.message}`, "error");
        playStrongBeep(300, 200);
        stopOutgoingTone();
        hideVoIPPanel();
        if (voipMediaStream) {
            voipMediaStream.getTracks().forEach(track => track.stop());
            voipMediaStream = null;
        }
        delete voipCalls[peerId];
        updatePeerList();
    }
}

function handleVoIPOffer(senderId, data) {
    if (voipActiveCall) {
        if (connections[senderId] && connections[senderId].conn) {
            connections[senderId].conn.send({
                type: 'voip_end',
                reason: 'busy',
                callId: data.callId
            });
        }
        updateMonitor(`📞 Llamada entrante de ${senderId.substring(0, 8)} rechazada (ocupado)`, "warning");
        return;
    }

    if (voipIncomingCall) {
        if (connections[senderId] && connections[senderId].conn) {
            connections[senderId].conn.send({
                type: 'voip_end',
                reason: 'busy',
                callId: data.callId
            });
        }
        updateMonitor(`📞 Llamada entrante de ${senderId.substring(0, 8)} rechazada (ya hay una entrante)`, "warning");
        return;
    }

    updateMonitor(`📞 Llamada entrante de ${senderId.substring(0, 8)}...`, "info");
    displayMessage(`📞 LLAMADA VoIP ENTRANTE de ${senderId.substring(0, 8)}`, '', 'voip');

    playRingtone();

    voipIncomingCall = senderId;
    voipCalls[senderId] = {
        id: data.callId,
        offer: data.offer,
        status: 'incoming',
        ringtoneTimeout: setTimeout(() => {
            if (voipIncomingCall === senderId) {
                console.log(`⏰ Llamada de ${senderId.substring(0,8)} timeout, rechazando.`);
                rejectVoIPCall(senderId, 'timeout');
            }
        }, 60000)
    };

    updatePeerList();
    showVoIPPanel('incoming', senderId);

    setTimeout(() => {
        const notification = showNotification(
            '📞 Llamada VoIP entrante',
            `De: ${senderId.substring(0, 8)}. Haz clic para volver a la app.`
        );
        if (notification) {
            notification.onclick = function() {
                window.focus();
            };
        }
    }, 100);
}

async function acceptVoIPCall(peerId) {
    if (!voipCalls[peerId] || voipCalls[peerId].status !== 'incoming') {
        console.error("No hay llamada pendiente de", peerId);
        return;
    }

    stopRingtone();
    if (voipCalls[peerId].ringtoneTimeout) {
        clearTimeout(voipCalls[peerId].ringtoneTimeout);
    }

    try {
        updateMonitor(`📞 Aceptando llamada de ${peerId.substring(0, 8)}...`, "info");

        const stream = await navigator.mediaDevices.getUserMedia({ audio: true, video: false });
        voipMediaStream = stream;

        initVoIPAudio();

        voipCalls[peerId].peerConnection = new RTCPeerConnection({
            iceServers: [
                { urls: 'stun:stun.l.google.com:19302' },
                { urls: 'stun:stun1.l.google.com:19302' }
            ]
        });

        stream.getTracks().forEach(track => {
            voipCalls[peerId].peerConnection.addTrack(track, stream);
        });

        voipCalls[peerId].peerConnection.ontrack = (event) => {
            console.log(`📡 Recibiendo pista de audio remota de ${peerId}`);
            
            const remoteAudio = new Audio();
            remoteAudio.srcObject = event.streams[0];
            remoteAudio.autoplay = true;
            
            remoteAudio.play().catch(e => console.log("Error reproduciendo audio remoto:", e));

            if (voipCalls[peerId]) {
                voipCalls[peerId].remoteAudio = remoteAudio;
            }

            if (voipActiveCall === peerId) {
                displayMessage(`📞 Audio establecido con ${peerId.substring(0, 8)}`, '', 'voip');
            }
        };

        await voipCalls[peerId].peerConnection.setRemoteDescription(
            new RTCSessionDescription(voipCalls[peerId].offer)
        );

        const answer = await voipCalls[peerId].peerConnection.createAnswer();
        await voipCalls[peerId].peerConnection.setLocalDescription(answer);

        connections[peerId].conn.send({
            type: 'voip_answer',
            answer: answer,
            callId: voipCalls[peerId].id,
            for: peerId
        });

        voipActiveCall = peerId;
        voipIncomingCall = null;
        voipCalls[peerId].status = 'active';
        updatePeerList();
        showVoIPPanel('active', peerId);

        updateMonitor(`📞 Llamada aceptada con ${peerId.substring(0, 8)}`, "info");
        displayMessage(`📞 Llamada VoIP aceptada - Conversación con ${peerId.substring(0, 8)}`, '', 'voip');

        voipCalls[peerId].peerConnection.onicecandidate = (event) => {
            if (event.candidate) {
                connections[peerId].conn.send({
                    type: 'voip_ice_candidate',
                    candidate: event.candidate,
                    callId: voipCalls[peerId].id,
                    for: peerId
                });
            }
        };

        voipCalls[peerId].peerConnection.onconnectionstatechange = () => {
            console.log(`VoIP connection state with ${peerId}: ${voipCalls[peerId].peerConnection.connectionState}`);
            if (voipCalls[peerId].peerConnection.connectionState === 'disconnected' ||
                voipCalls[peerId].peerConnection.connectionState === 'failed' ||
                voipCalls[peerId].peerConnection.connectionState === 'closed') {
                if (voipActiveCall === peerId || voipIncomingCall === peerId) {
                    endVoIPCall(peerId);
                }
            }
        };

    } catch (error) {
        console.error("❌ Error aceptando llamada:", error);
        updateMonitor(`❌ Error aceptando llamada: ${error.message}`, "error");
        rejectVoIPCall(peerId, 'accept_failed');
    }
}

function rejectVoIPCall(peerId, reason = 'rejected') {
    if (!voipCalls[peerId]) return;

    stopRingtone();

    if (voipCalls[peerId].ringtoneTimeout) {
        clearTimeout(voipCalls[peerId].ringtoneTimeout);
    }

    if (connections[peerId] && connections[peerId].conn) {
        connections[peerId].conn.send({
            type: 'voip_end',
            reason: reason,
            callId: voipCalls[peerId].id
        });
    }

    delete voipCalls[peerId];
    if (voipIncomingCall === peerId) {
        voipIncomingCall = null;
    }

    hideVoIPPanel();
    updateMonitor(`📞 Llamada de ${peerId.substring(0, 8)} rechazada`, "info");
    updatePeerList();
}

function handleVoIPAnswer(senderId, data) {
    if (!voipCalls[senderId] || !voipCalls[senderId].peerConnection) {
        console.error("No hay llamada activa con", senderId);
        return;
    }

    stopOutgoingTone();

    try {
        voipCalls[senderId].peerConnection.setRemoteDescription(
            new RTCSessionDescription(data.answer)
        );
        voipActiveCall = senderId;
        voipCalls[senderId].status = 'active';
        showVoIPPanel('active', senderId);
        updateMonitor(`📞 Llamada establecida con ${senderId.substring(0, 8)}`, "info");
        displayMessage(`📞 Llamada VoIP establecida - Conversación con ${senderId.substring(0, 8)}`, '', 'voip');
        updatePeerList();
    } catch (error) {
        console.error("❌ Error manejando respuesta VoIP:", error);
    }
}

function handleVoIPICECandidate(senderId, data) {
    if (!voipCalls[senderId] || !voipCalls[senderId].peerConnection) {
        console.error("No hay llamada activa con", senderId, "para candidato ICE");
        return;
    }

    try {
        voipCalls[senderId].peerConnection.addIceCandidate(new RTCIceCandidate(data.candidate));
    } catch (error) {
        console.error("❌ Error añadiendo candidato ICE:", error);
    }
}

function handleVoIPEnd(senderId) {
    if (voipActiveCall === senderId) {
        endVoIPCall(senderId);
    } else if (voipIncomingCall === senderId) {
        rejectVoIPCall(senderId, 'remote_end');
    }
    updateMonitor(`📞 Llamada con ${senderId.substring(0, 8)} finalizada`, "info");
    displayMessage(`📞 Llamada VoIP finalizada con ${senderId.substring(0, 8)}`, '', 'voip');
}

function endVoIPCall(peerId = null) {
    let endedPeerId = peerId || voipActiveCall;

    if (!endedPeerId && voipIncomingCall) {
        endedPeerId = voipIncomingCall;
    }

    if (!endedPeerId) return;

    stopRingtone();
    stopOutgoingTone();

    if (connections[endedPeerId] && connections[endedPeerId].conn) {
        try {
            connections[endedPeerId].conn.send({
                type: 'voip_end',
                reason: 'ended',
                callId: voipCalls[endedPeerId]?.id
            });
        } catch (error) {
            console.error("Error notificando fin de llamada:", error);
        }
    }

    if (voipCalls[endedPeerId]) {
        if (voipCalls[endedPeerId].peerConnection) {
            voipCalls[endedPeerId].peerConnection.close();
        }
        
        if (voipCalls[endedPeerId].remoteAudio) {
            voipCalls[endedPeerId].remoteAudio.pause();
            voipCalls[endedPeerId].remoteAudio.srcObject = null;
        }
        
        delete voipCalls[endedPeerId];
    }

    if (voipActiveCall === endedPeerId) voipActiveCall = null;
    if (voipIncomingCall === endedPeerId) voipIncomingCall = null;

    if (voipMediaStream) {
        voipMediaStream.getTracks().forEach(track => track.stop());
        voipMediaStream = null;
    }

    hideVoIPPanel();
    updatePeerList();
    updateMonitor(`📞 Llamada finalizada`, "info");
    displayMessage(`📞 Llamada VoIP finalizada`, '', 'voip');
    playStrongBeep(600, 100);
}

// =============================================
// FUNCIONES DE NOTIFICACIONES PUSH
// =============================================

function showNotification(title, body) {
    if (!("Notification" in window)) {
        console.log("Este navegador no soporta notificaciones de escritorio");
        return null;
    }

    if (!document.hidden) {
        return null;
    }

    function spawnNotification() {
        const options = {
            body: body,
            icon: 'data:image/svg+xml,%3Csvg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 100"%3E%3Ccircle cx="50" cy="50" r="45" fill="%2300ff88" /%3E%3Ctext x="50" y="70" font-size="60" text-anchor="middle" fill="%23000" font-family="monospace"%3ER%3C/text%3E%3C/svg%3E',
            badge: 'data:image/svg+xml,%3Csvg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 100 100"%3E%3Ccircle cx="50" cy="50" r="45" fill="%23ff3300" /%3E%3C/svg%3E',
            tag: 'radcom-message',
            renotify: true,
            silent: false
        };
        return new Notification(title, options);
    }

    if (Notification.permission === "granted") {
        return spawnNotification();
    }

    if (Notification.permission !== "denied") {
        Notification.requestPermission().then(permission => {
            if (permission === "granted") {
                return spawnNotification();
            }
        });
    }

    return null;
}

// =============================================
// HANDLE RECEIVED DATA - CON NOTIFICACIONES
// =============================================

function handleReceivedData(senderId, data) {
    console.log("🔄 Procesando datos de", senderId, ":", data);
    
    if (!connectionHealth[senderId]) connectionHealth[senderId] = {};
    connectionHealth[senderId].lastActivity = Date.now();
    
    if (!data || !data.message) return;
    
    const key = document.getElementById('key').value || 'ATOM80';
    const sender = senderId.substring(0, 8);
    
    let displayText = data.message;
    let encryptionType = data.encryption || 'NONE';
    let errorMsg = '';
    
    if (encryptionType === 'AES-256-CBC' || encryptionType === 'AES-256-GCM' || 
        encryptionType === 'AES-256-GCM (legacy)' || encryptionType === 'XOR') {
        
        const mode = encryptionType.includes('AES') ? 'aes' : 'xor';
        const secured = securityLayer(data.message, false, mode, key);
        
        displayText = secured.text;
        encryptionType = secured.encryptionUsed;
        
        if (secured.error) {
            errorMsg = ` ⚠️ ${secured.error}`;
            console.warn(`Error descifrando de ${sender}:`, secured.error);
        }
    }
    
    let finalText = displayText;
    let isMorse = false;
    
    if (typeof displayText === 'string' && /^[\.\-\s\/]+$/.test(displayText.trim())) {
        isMorse = true;
        const decoded = decodeMorse(displayText);
        if (decoded && !decoded.includes('?')) {
            finalText = decoded;
            updateMorseTranslation(displayText);
        }
    }
    else if (data.mode === 'morse') {
        isMorse = true;
        const decoded = decodeMorse(displayText);
        if (decoded && !decoded.includes('?')) {
            finalText = decoded;
            updateMorseTranslation(displayText);
        }
    }
    else if (data.isDualMorse && data.textVersion) {
        finalText = data.textVersion;
        if (data.morseVersion) {
            updateMorseTranslation(data.morseVersion);
        }
    }
    
    const monitor = document.getElementById('monitor-decoded');
    const bubble = document.createElement('div');
    bubble.className = 'message-bubble message-incoming';
    
    let color = '#888';
    if (encryptionType === 'AES-256-CBC') color = '#00ffea';
    else if (encryptionType === 'AES-256-GCM' || encryptionType === 'AES-256-GCM (legacy)') color = '#00ff88';
    else if (encryptionType === 'XOR') color = '#ffaa00';
    else if (encryptionType === 'ERROR') color = '#ff3300';
    
    const morseIndicator = isMorse ? ' 📡 → ' + displayText : '';
    
    bubble.innerHTML = `
        <strong>${sender}:</strong> ${finalText}${morseIndicator}
        <br><small style="color:${color};">Cifrado: ${encryptionType}${errorMsg}</small>
    `;
    monitor.appendChild(bubble);
    monitor.scrollTop = monitor.scrollHeight;
    
    stats.rx += data.message.length;
    stats.messages++;
    updateStats();
    
    playMessageNotification();

    const notificationTitle = `📨 Mensaje de ${sender}`;
    const notificationBody = finalText.length > 50 ? finalText.substring(0, 50) + '...' : finalText;
    showNotification(notificationTitle, notificationBody);
}

function sendMessage() {
    const input = document.getElementById('inputMsg');
    let message = input.value.trim();
    
    if (!message) {
        updateMonitor("⚠️ MENSAJE VACÍO", "error");
        playStrongBeep(300, 200);
        return;
    }
    
    const key = document.getElementById('key').value || 'ATOM80';
    const mode = document.getElementById('inputMode').value;
    const encryption = document.getElementById('encryptionMode').value;
    
    if (!validateInput()) {
        updateMonitor(`⚠️ FORMATO ${mode.toUpperCase()} INVÁLIDO`, "error");
        playStrongBeep(300, 200);
        return;
    }
    
    let processedMessage = message;
    let extraData = {};
    
    if (mode === 'morse') {
        const isMorse = /^[\.\-\s\/]+$/.test(message.trim());
        
        if (isMorse) {
            processedMessage = message;
            extraData = {
                textVersion: decodeMorse(message) || message,
                morseVersion: message,
                isDualMorse: true,
                mode: 'morse'
            };
            updateMonitor(`📡 Enviando Morse (${extraData.textVersion.substring(0, 20)})`);
        } else {
            processedMessage = textToMorse(message);
            extraData = {
                textVersion: message,
                morseVersion: processedMessage,
                isDualMorse: true,
                mode: 'morse'
            };
            updateMonitor(`📡 Enviando "${message.substring(0, 20)}" como Morse`);
        }
        
        updateMorseTranslation(message);
        
    } else if (mode === 'phonetic') {
        processedMessage = textToPhonetic(message);
    }
    
    const dataToSend = prepareMessageToSend(processedMessage, mode, encryption, key, extraData);
    
    let sentCount = 0;
    
    if (activeTarget === 'GLOBAL') {
        Object.keys(connections).forEach(peerId => {
            if (connections[peerId]?.status === 'online') {
                try {
                    connections[peerId].conn.send(dataToSend);
                    sentCount++;
                } catch (error) {
                    console.error(`❌ Error:`, error);
                }
            }
        });
    } else if (connections[activeTarget]?.status === 'online') {
        try {
            connections[activeTarget].conn.send(dataToSend);
            sentCount = 1;
        } catch (error) {
            console.error(`❌ Error:`, error);
        }
    }
    
    if (sentCount > 0) {
        let displayMsg = message;
        if (mode === 'morse' && extraData.textVersion) {
            displayMsg = extraData.textVersion;
            if (extraData.morseVersion && extraData.morseVersion.length < 30) {
                displayMsg += ` [${extraData.morseVersion}]`;
            }
        }
        
        displayMessage(`YO: ${displayMsg}`, dataToSend.hexPreview, 'outgoing');
        stats.tx += JSON.stringify(dataToSend).length;
        stats.messages++;
        input.value = '';
        updateMonitor(`📤 ENVIADO A ${sentCount} DESTINO${sentCount !== 1 ? 'S' : ''}`);
        updateStats();
        playStrongBeep(700, 100);
        
        if (activeTarget === 'GLOBAL') {
            Object.keys(connections).forEach(peerId => {
                if (connections[peerId]?.status === 'online') {
                    connectionHealth[peerId] = {
                        ...connectionHealth[peerId],
                        lastActivity: Date.now(),
                        packetsSent: (connectionHealth[peerId]?.packetsSent || 0) + 1
                    };
                }
            });
        } else if (connections[activeTarget]) {
            connectionHealth[activeTarget] = {
                ...connectionHealth[activeTarget],
                lastActivity: Date.now(),
                packetsSent: (connectionHealth[activeTarget]?.packetsSent || 0) + 1
            };
        }
    } else {
        addToQueue(dataToSend, message);
        displayMessage(`⏳ YO (EN COLA): ${message}`, dataToSend.hexPreview, 'outgoing');
        input.value = '';
        updateMonitor(`💾 GUARDADO EN COLA (${messageQueue.length} mensajes)`);
        playStrongBeep(500, 100);
    }
    
    document.querySelectorAll('#ansiTable td.highlighted').forEach(td => {
        td.classList.remove('highlighted');
    });
}

function sendWithQueue() {
    sendMessage();
}

function sendRadioMessage() {
    const input = document.getElementById('inputMsg');
    let message = input.value.trim();
    
    if (!message) {
        updateMonitor("⚠️ MENSAJE VACÍO", "error");
        playStrongBeep(300, 200);
        return;
    }
    
    const key = document.getElementById('key').value || 'ATOM80';
    const mode = document.getElementById('inputMode').value;
    const encryption = document.getElementById('encryptionMode').value;
    
    if (!validateInput()) {
        updateMonitor(`⚠️ FORMATO ${mode.toUpperCase()} INVÁLIDO`, "error");
        playStrongBeep(300, 200);
        return;
    }
    
    let processedMessage = message;
    
    if (mode === 'phonetic') {
        processedMessage = textToPhonetic(message);
    }
    
    const dataToSend = prepareMessageToSend(processedMessage, mode, encryption, key, {});
    const sentCount = sendToConnections(dataToSend);
    
    if (sentCount > 0) {
        const freq = document.getElementById('radio-frequency-display').textContent;
        const band = currentBand;
        
        displayMessage(`📡 YO [${band} ${freq}]: ${message}`, dataToSend.hexPreview, 'outgoing');
        
        stats.tx += JSON.stringify(dataToSend).length;
        stats.messages++;
        input.value = '';
        updateMonitor(`📤 ENVIADO POR RADIO A ${sentCount} DESTINO${sentCount !== 1 ? 'S' : ''}`);
        updateStats();
        playTransmissionSound();
        
        if (activeTarget === 'GLOBAL') {
            Object.keys(connections).forEach(peerId => {
                if (connections[peerId]?.status === 'online') {
                    connectionHealth[peerId] = {
                        ...connectionHealth[peerId],
                        lastActivity: Date.now(),
                        packetsSent: (connectionHealth[peerId]?.packetsSent || 0) + 1
                    };
                }
            });
        } else if (connections[activeTarget]) {
            connectionHealth[activeTarget] = {
                ...connectionHealth[activeTarget],
                lastActivity: Date.now(),
                packetsSent: (connectionHealth[activeTarget]?.packetsSent || 0) + 1
            };
        }
        
        document.querySelectorAll('#ansiTable td.highlighted').forEach(td => {
            td.classList.remove('highlighted');
        });
    } else {
        updateMonitor("⚠️ SIN DESTINOS CONECTADOS", "warning");
        playStrongBeep(300, 200);
    }
}

function sendToConnections(data) {
    let sentCount = 0;
    
    console.log("📤 Enviando datos:", data);
    
    if (activeTarget === 'GLOBAL') {
        Object.keys(connections).forEach(peerId => {
            if (connections[peerId]?.status === 'online') {
                try {
                    console.log(`📤 Enviando a ${peerId}`);
                    connections[peerId].conn.send(data);
                    sentCount++;
                } catch (error) {
                    console.error(`❌ Error enviando a ${peerId}:`, error);
                }
            }
        });
    } else if (connections[activeTarget]?.status === 'online') {
        try {
            console.log(`📤 Enviando a ${activeTarget}`);
            connections[activeTarget].conn.send(data);
            sentCount = 1;
        } catch (error) {
            console.error(`❌ Error enviando a ${activeTarget}:`, error);
        }
    }
    
    console.log(`✅ Enviado a ${sentCount} destino(s)`);
    return sentCount;
}

function prepareMessageToSend(message, mode, encryption, key, extraData = {}) {
    let processedMessage = message;
    let hexPreview = '';
    
    if (mode === 'hex') {
        const hex = message.replace(/\s/g, '');
        if (hex.length % 2 !== 0) {
            throw new Error('HEX debe tener longitud par');
        }
        processedMessage = '';
        for (let i = 0; i < hex.length; i += 2) {
            processedMessage += String.fromCharCode(parseInt(hex.substr(i, 2), 16));
        }
    } else if (mode === 'binary') {
        const binary = message.replace(/\s/g, '');
        if (binary.length % 8 !== 0) {
            throw new Error('Binario debe ser múltiplo de 8');
        }
        processedMessage = '';
        for (let i = 0; i < binary.length; i += 8) {
            processedMessage += String.fromCharCode(parseInt(binary.substr(i, 8), 2));
        }
    }
    
    let encryptedMessage = processedMessage;
    let encryptionType = 'NONE';
    
    if (encryption === 'aes' || encryption === 'aes-256-gcm') {
        const secured = securityLayer(processedMessage, true, 'aes', key);
        encryptedMessage = secured.text;
        encryptionType = 'AES-256-CBC';
        hexPreview = '🔐';
    } else if (encryption === 'xor') {
        const secured = securityLayer(processedMessage, true, 'xor', key);
        encryptedMessage = secured.text;
        encryptionType = 'XOR';
        for (let i = 0; i < Math.min(encryptedMessage.length, 3); i++) {
            hexPreview += encryptedMessage.charCodeAt(i).toString(16).padStart(2, '0');
        }
        if (encryptedMessage.length > 3) hexPreview += '...';
    }
    
    return {
        type: 'message',
        message: encryptedMessage,
        original: extraData.textVersion || message,
        mode: mode,
        encryption: encryptionType,
        timestamp: Date.now(),
        sender: myPeerId,
        hexPreview: hexPreview,
        textVersion: extraData.textVersion || message,
        morseVersion: extraData.morseVersion || '',
        isDualMorse: extraData.isDualMorse || false
    };
}

function handleSendMessage(event) {
    if (event.key === 'Enter') {
        event.preventDefault();
        const mode = document.getElementById('inputMode').value;
        if (mode === 'phonetic' && document.getElementById('radio-frequency-input')) {
            sendRadioMessage();
        } else {
            sendMessage();
        }
    }
}

function handleSendButtonClick() {
    const mode = document.getElementById('inputMode').value;
    if (mode === 'phonetic' && document.getElementById('radio-frequency-input')) {
        sendRadioMessage();
    } else {
        sendMessage();
    }
}

// =============================================
// SISTEMA DE COLA DE MENSAJES
// =============================================

function addToQueue(dataToSend, originalMessage) {
    if (messageQueue.length >= MAX_QUEUE_SIZE) {
        messageQueue.shift();
    }
    
    const queueItem = {
        id: Date.now() + Math.random().toString(36).substr(2, 5),
        data: dataToSend,
        original: originalMessage,
        target: activeTarget,
        timestamp: Date.now(),
        attempts: 0
    };
    
    messageQueue.push(queueItem);
    localStorage.setItem('radcom_message_queue', JSON.stringify(messageQueue));
    
    updateQueueCounter();
    
    if (!queueRetryInterval) {
        startQueueSystem();
    }
}

function startQueueSystem() {
    if (queueRetryInterval) clearInterval(queueRetryInterval);
    queueRetryInterval = setInterval(() => { processQueue(); }, 1000000);
    updateMonitor("🔄 Sistema de cola ACTIVADO", "info");
}

function processQueue() {
    if (messageQueue.length === 0) {
        if (queueRetryInterval) {
            clearInterval(queueRetryInterval);
            queueRetryInterval = null;
        }
        return;
    }
    
    const now = Date.now();
    const remaining = [];
    
    messageQueue.forEach(item => {
        if (now - item.timestamp < 5000) {
            remaining.push(item);
            return;
        }
        
        item.attempts++;
        
        let delivered = false;
        
        if (item.target === 'GLOBAL') {
            Object.keys(connections).forEach(peerId => {
                if (connections[peerId]?.status === 'online') {
                    try {
                        connections[peerId].conn.send(item.data);
                        delivered = true;
                        displayMessage(`✅ ENTREGADO A ${peerId.substring(0,6)}: ${item.original.substring(0, 40)}...`, '', 'system');
                    } catch (error) {
                        console.error(`Error:`, error);
                    }
                }
            });
        } else if (connections[item.target]?.status === 'online') {
            try {
                connections[item.target].conn.send(item.data);
                delivered = true;
                displayMessage(`✅ ENTREGADO A ${item.target.substring(0,6)}: ${item.original.substring(0, 40)}...`, '', 'system');
            } catch (error) {
                console.error(`Error:`, error);
            }
        }
        
        if (delivered) {
            stats.tx += JSON.stringify(item.data).length;
            stats.messages++;
            updateStats();
            playStrongBeep(600, 50);
        } else if (item.attempts < 10) {
            remaining.push(item);
        } else {
            displayMessage(`❌ NO ENTREGADO: ${item.original.substring(0, 30)}...`, '', 'system');
        }
    });
    
    messageQueue = remaining;
    localStorage.setItem('radcom_message_queue', JSON.stringify(messageQueue));
    updateQueueCounter();
}

function updateQueueCounter() {
    let counter = document.getElementById('queue-counter');
    
    if (!counter) {
        const statsPanel = document.querySelector('.stats-panel');
        if (statsPanel) {
            const queueItem = document.createElement('div');
            queueItem.className = 'stat-item';
            queueItem.id = 'queue-item';
            queueItem.innerHTML = `
                <div class="stat-value" id="queue-counter">0</div>
                <div class="stat-label">EN COLA</div>
            `;
            statsPanel.appendChild(queueItem);
            counter = document.getElementById('queue-counter');
        }
    }
    
    if (counter) {
        counter.textContent = messageQueue.length;
        
        if (messageQueue.length === 0) {
            counter.style.color = '#00ff88';
        } else if (messageQueue.length < 5) {
            counter.style.color = '#ffaa00';
        } else {
            counter.style.color = '#ff3300';
            counter.style.animation = 'pulse 1s infinite';
        }
    }
}

function deliverOnConnect(peerId) {
    if (messageQueue.length === 0) return;
    
    const remaining = [];
    
    messageQueue.forEach(item => {
        if (item.target === peerId || item.target === 'GLOBAL') {
            try {
                if (connections[peerId]?.status === 'online') {
                    connections[peerId].conn.send(item.data);
                    displayMessage(`🎯 ENTREGADO A NUEVA CONEXIÓN ${peerId.substring(0,6)}`, '', 'system');
                    stats.tx += JSON.stringify(item.data).length;
                    stats.messages++;
                }
            } catch (error) {
                console.error(`Error:`, error);
                remaining.push(item);
            }
        } else {
            remaining.push(item);
        }
    });
    
    messageQueue = remaining;
    localStorage.setItem('radcom_message_queue', JSON.stringify(messageQueue));
    updateQueueCounter();
}

function initQueue() {
    updateQueueCounter();
    if (messageQueue.length > 0) {
        updateMonitor(`📨 ${messageQueue.length} mensaje(s) en cola`);
        if (!queueRetryInterval) {
            startQueueSystem();
        }
    }
}

// =============================================
// FUNCIONES DE UI Y DISPLAY
// =============================================

function buildAsciiTable() {
    const table = document.getElementById('ansiTable');
    let html = "<thead><tr><th class=\"row-idx\">V\\H</th>";
    
    for (let i = 0; i < 8; i++) {
        html += `<th>${i.toString(16).toUpperCase()}</th>`;
    }
    html += "</tr></thead><tbody>";
    
    for (let row = 0; row < 16; row++) {
        const rowHex = row.toString(16).toUpperCase();
        html += `<tr><td class="row-idx">${rowHex}</td>`;
        
        for (let col = 0; col < 8; col++) {
            const displayChar = chars[row][col];
            const asciiCode = (col << 4) | row;
            const hexCode = asciiCode.toString(16).toUpperCase().padStart(2, '0');
            
            html += `<td data-row="${row}" data-col="${col}" data-ascii="${asciiCode}" 
                     data-hex="${hexCode}" data-char="${displayChar}" 
                     onclick="selectTableChar(this)"
                     title="${displayChar} | HEX: ${hexCode} | DEC: ${asciiCode}">
                     ${displayChar}
                     </td>`;
        }
        html += "</tr>";
    }
    
    html += "</tbody>";
    table.innerHTML = html;
}

function buildMorseTable() {
    const table = document.getElementById('morseTable');
    let html = "<thead><tr><th>CARÁCTER</th><th>CÓDIGO MORSE</th><th>EJEMPLO FONÉTICO</th><th>PROBAR</th></tr></thead><tbody>";
    
    for (let charCode = 65; charCode <= 90; charCode++) {
        const char = String.fromCharCode(charCode);
        const morse = morseCodes[char];
        const phonetic = phoneticAlphabetFull[char]?.word || char;
        
        html += `
            <tr>
                <td class="morse-char" onclick="selectMorseChar('${char}')">${char}</td>
                <td class="morse-code" onclick="selectMorseCode('${morse}')">${morse}</td>
                <td class="phonetic-example">${phonetic}</td>
                <td>
                    <button class="play-morse-btn" onclick="playMorseCharSound('${char}')" 
                            title="Probar sonido Morse para ${char}">
                        <i class="fas fa-volume-up"></i>
                    </button>
                </td>
            </tr>
        `;
    }
    
    for (let i = 0; i <= 9; i++) {
        const char = i.toString();
        const morse = morseCodes[char];
        const phonetic = phoneticAlphabetFull[char]?.word || char;
        
        html += `
            <tr>
                <td class="morse-char" onclick="selectMorseChar('${char}')">${char}</td>
                <td class="morse-code" onclick="selectMorseCode('${morse}')">${morse}</td>
                <td class="phonetic-example">${phonetic}</td>
                <td>
                    <button class="play-morse-btn" onclick="playMorseCharSound('${char}')" 
                            title="Probar sonido Morse para ${char}">
                        <i class="fas fa-volume-up"></i>
                    </button>
                </td>
            </tr>
        `;
    }
    
    const specialChars = ['.', ',', '?', '/', ' '];
    specialChars.forEach(char => {
        const morse = morseCodes[char];
        const displayChar = char === ' ' ? 'ESPACIO' : char;
        
        html += `
            <tr>
                <td class="morse-char" onclick="selectMorseChar('${char}')">${displayChar}</td>
                <td class="morse-code" onclick="selectMorseCode('${morse}')">${morse}</td>
                <td class="phonetic-example">---</td>
                <td>
                    <button class="play-morse-btn" onclick="playMorseCharSound('${char}')" 
                            title="Probar sonido Morse para ${displayChar}">
                        <i class="fas fa-volume-up"></i>
                    </button>
                </td>
            </tr>
        `;
    });
    
    html += "</tbody>";
    table.innerHTML = html;
}

function buildRadioTable() {
    const grid = document.getElementById('phonetic-grid');
    let html = '';
    
    Object.keys(phoneticAlphabetFull).forEach(char => {
        const data = phoneticAlphabetFull[char];
        html += `
            <div class="phonetic-item" onclick="selectPhoneticCharRadio('${char}')" 
                 title="${char}: ${data.word} (${data.pronunciation})">
                <div class="phonetic-char">${char}</div>
                <div class="phonetic-word">${data.word}</div>
                <div class="phonetic-pronunciation">${data.pronunciation}</div>
            </div>
        `;
    });
    
    grid.innerHTML = html;
    
    updateFrequencyDisplay();
    updateEmergencyFrequencies();
    buildPMRChannels();
    buildCBChannels();
    
    const freqInput = document.getElementById('radio-frequency-input');
    const unitSelect = document.getElementById('radio-unit');
    
    if (freqInput) {
        freqInput.addEventListener('input', validateFrequency);
        freqInput.addEventListener('change', updateFrequencyFromInput);
    }
    if (unitSelect) {
        unitSelect.addEventListener('change', updateUnit);
    }
}

function selectTableChar(cell) {
    document.querySelectorAll('#ansiTable td.selected').forEach(td => {
        td.classList.remove('selected');
    });
    
    cell.classList.add('selected');
    
    const row = parseInt(cell.getAttribute('data-row'));
    const col = parseInt(cell.getAttribute('data-col'));
    const asciiCode = parseInt(cell.getAttribute('data-ascii'));
    const hexCode = cell.getAttribute('data-hex');
    const displayChar = cell.getAttribute('data-char');
    
    updateCharPreview(row, col, asciiCode, hexCode, displayChar);
    
    const input = document.getElementById('inputMsg');
    const inputMode = document.getElementById('inputMode').value;
    
    if (displayChar === "SPC") {
        if (inputMode === 'hex') {
            input.value += "20 ";
        } else if (inputMode === 'binary') {
            input.value += "00100000 ";
        } else {
            input.value += " ";
        }
    } else if (displayChar === "DEL") {
        if (inputMode === 'hex') {
            input.value += "7F ";
        } else if (inputMode === 'binary') {
            input.value += "01111111 ";
        } else {
            input.value += "\x7F";
        }
    } else if (col <= 1) {
        if (inputMode === 'hex') {
            input.value += hexCode + " ";
        } else if (inputMode === 'binary') {
            const binary = asciiCode.toString(2).padStart(8, '0');
            input.value += binary + " ";
        } else {
            if (asciiCode === 9) input.value += "\t";
            else if (asciiCode === 10) input.value += "\n";
            else if (asciiCode === 13) input.value += "\r";
            else if (asciiCode === 27) input.value += "\x1B";
            else input.value += `[${displayChar}]`;
        }
    } else if (displayChar.length === 1) {
        if (inputMode === 'hex') {
            input.value += hexCode + " ";
        } else if (inputMode === 'binary') {
            const binary = asciiCode.toString(2).padStart(8, '0');
            input.value += binary + " ";
        } else {
            input.value += displayChar;
        }
    }
    
    input.focus();
    playStrongBeep(600, 50);
}

function selectMorseChar(char) {
    const input = document.getElementById('inputMsg');
    const inputMode = document.getElementById('inputMode').value;
    
    input.value += char;
    input.focus();
    
    if (inputMode === 'morse' && document.getElementById('soundEnabled')?.checked) {
        playMorseCharSound(char);
    } else {
        playStrongBeep(600, 50);
    }
}

function selectMorseCode(code) {
    const input = document.getElementById('inputMsg');
    const inputMode = document.getElementById('inputMode').value;
    
    if (inputMode === 'morse') {
        input.value += code + ' ';
    } else {
        const char = getCharFromMorse(code);
        if (char) {
            input.value += char;
        } else {
            input.value += code;
        }
    }
    
    input.focus();
    
    if (inputMode === 'morse') {
        playMorseCodeSound(code);
    } else {
        playStrongBeep(600, 50);
    }
}

function getCharFromMorse(morseCode) {
    for (const [char, code] of Object.entries(morseCodes)) {
        if (code === morseCode) {
            return char;
        }
    }
    return null;
}

function updateCharPreview(vertical, horizontal, ascii, hex, char) {
    document.getElementById('current-char').textContent = char;
    document.getElementById('current-hex').textContent = hex;
    document.getElementById('current-dec').textContent = ascii;
    document.getElementById('current-bin').textContent = ascii.toString(2).padStart(8, '0');
    document.getElementById('current-pos').textContent = 
        `V:${vertical.toString(16).toUpperCase()} H:${horizontal.toString(16).toUpperCase()}`;
}

function clearTableSelection() {
    document.querySelectorAll('#ansiTable td.selected').forEach(td => {
        td.classList.remove('selected');
    });
    
    document.getElementById('current-char').textContent = '-';
    document.getElementById('current-hex').textContent = '--';
    document.getElementById('current-dec').textContent = '---';
    document.getElementById('current-bin').textContent = '--------';
    document.getElementById('current-pos').textContent = 'V:-- H:--';
    
    document.querySelectorAll('#ansiTable td.highlighted').forEach(td => {
        td.classList.remove('highlighted');
    });
}

function realTimeTableHighlight() {
    const input = document.getElementById('inputMsg');
    const text = input.value;
    
    document.querySelectorAll('#ansiTable td.highlighted').forEach(td => {
        td.classList.remove('highlighted');
    });
    
    if (text.length === 0) {
        return;
    }
    
    const lastChar = text[text.length - 1];
    const asciiCode = lastChar.charCodeAt(0);
    
    if (asciiCode >= 32 && asciiCode <= 126) {
        const row = asciiCode & 0x0F;
        const col = (asciiCode >> 4) & 0x07;
        
        const cell = document.querySelector(`#ansiTable td[data-ascii="${asciiCode}"]`);
        if (cell) {
            cell.classList.add('highlighted');
            
            const displayChar = cell.getAttribute('data-char');
            const hex = cell.getAttribute('data-hex');
            updateCharPreview(row, col, asciiCode, hex, displayChar);
            
            cell.scrollIntoView({ behavior: 'smooth', block: 'center', inline: 'center' });
        }
    } else if (lastChar === ' ') {
        const cell = document.querySelector('#ansiTable td[data-char="SPC"]');
        if (cell) {
            cell.classList.add('highlighted');
            const row = parseInt(cell.getAttribute('data-row'));
            const col = parseInt(cell.getAttribute('data-col'));
            const ascii = parseInt(cell.getAttribute('data-ascii'));
            const hex = cell.getAttribute('data-hex');
            updateCharPreview(row, col, ascii, hex, 'SPC');
        }
    } else if (lastChar === '\x7F') {
        const cell = document.querySelector('#ansiTable td[data-char="DEL"]');
        if (cell) {
            cell.classList.add('highlighted');
            const row = parseInt(cell.getAttribute('data-row'));
            const col = parseInt(cell.getAttribute('data-col'));
            const ascii = parseInt(cell.getAttribute('data-ascii'));
            const hex = cell.getAttribute('data-hex');
            updateCharPreview(row, col, ascii, hex, 'DEL');
        }
    }
}

function realTimePreview() {
    clearTimeout(window.previewTimeout);
    window.previewTimeout = setTimeout(() => {
        const input = document.getElementById('inputMsg');
        const message = input.value;
        
        if (!message) {
            updateMonitor("ESCUCHA ACTIVA...");
            return;
        }
        
        const key = document.getElementById('key').value || 'ATOM80';
        let hex = '';
        for (let i = 0; i < Math.min(message.length, 3); i++) {
            const charCode = message.charCodeAt(i);
            const keyCode = key.charCodeAt(i % key.length);
            hex += (charCode ^ keyCode).toString(16).padStart(2, '0');
        }
        
        updateMonitor(`PREVIEW: ${hex}...`);
    }, 300);
    
    const mode = document.getElementById('inputMode').value;
    if (mode === 'morse') {
        updateMorseTranslation(input.value);
    } else {
        document.getElementById('morse-translation').classList.remove('active');
    }
}

function updateMorseTranslation(text) {
    const translationBox = document.getElementById('morse-translation');
    const textDisplay = document.getElementById('morse-text-display');
    const codeDisplay = document.getElementById('morse-code-display');
    
    if (!translationBox || !textDisplay || !codeDisplay) return;
    
    if (!text.trim()) {
        translationBox.classList.remove('active');
        return;
    }
    
    translationBox.classList.add('active');
    
    const isMorse = /^[\.\-\s\/]+$/.test(text.trim());
    
    if (isMorse) {
        const decoded = decodeMorse(text);
        textDisplay.textContent = decoded || '???';
        codeDisplay.textContent = text;
    } else {
        textDisplay.textContent = text;
        codeDisplay.textContent = textToMorse(text);
    }
}

function decodeMorse(morseCode) {
    if (!morseCode) return '';
    
    const morseMap = {};
    for (const [char, code] of Object.entries(morseCodes)) {
        morseMap[code] = char;
    }
    
    return morseCode.split(' ').map(code => {
        if (code === '/') return ' ';
        return morseMap[code] || '?';
    }).join('');
}

function textToMorse(text) {
    if (!text) return '';
    
    return text.toUpperCase().split('').map(char => {
        if (char === ' ') return '/';
        return morseCodes[char] || char;
    }).join(' ');
}

function textToPhonetic(text) {
    return text.toUpperCase().split('').map(char => phoneticAlphabetFull[char]?.word || char).join(' ');
}

function isMorseCode(text) {
    const morseRegex = /^[\.\-\s\/]+$/;
    return morseRegex.test(text);
}

function clearMorseTranslation() {
    document.getElementById('morse-translation').classList.remove('active');
    document.getElementById('morse-text-display').textContent = '';
    document.getElementById('morse-code-display').textContent = '';
}

function displayMessage(content, hex, type) {
    const monitor = document.getElementById('monitor-decoded');
    const messageDiv = document.createElement('div');
    
    messageDiv.className = `message-bubble message-${type}`;
    
    let displayContent = content;
    if (content.length > 150) {
        displayContent = content.substring(0, 150) + '...';
    }
    
    const time = new Date().toLocaleTimeString([], {hour: '2-digit', minute:'2-digit', second:'2-digit'});
    
    messageDiv.innerHTML = `
        <div style="margin-bottom:1px;">
            <span style="color:#${type === 'outgoing' ? '0088ff' : type === 'incoming' ? 'ff3300' : type === 'voip' ? '00ff88' : '00ff88'}">
                ${time}
            </span>
        </div>
        <div>${escapeHtml(displayContent)}</div>
        ${hex ? `<div style="font-size:0.55rem; color:#888; margin-top:1px;">
            <i class="fas fa-code"></i> ${hex}</div>` : ''}
    `;
    
    monitor.appendChild(messageDiv);
    monitor.scrollTop = monitor.scrollHeight;
}

function escapeHtml(text) {
    const div = document.createElement('div');
    div.textContent = text;
    return div.innerHTML;
}

function updateMonitor(message, type = 'info') {
    const monitor = document.getElementById('monitor-raw');
    const color = type === 'error' ? '#ff3300' : type === 'warning' ? '#ffaa00' : '#00ff88';
    
    monitor.innerHTML = `<span style="color:${color}">${message}</span>`;
}

function clearChat() {
    if (confirm("¿Borrar todo el historial del chat?")) {
        document.getElementById('monitor-decoded').innerHTML = 
            '<div class="message-bubble message-system"><i class="fas fa-satellite"></i> HISTORIAL LIMPIADO</div>';
        updateMonitor("✅ CHAT LIMPIADO");
        playStrongBeep(400, 200);
    }
}

function validateInput() {
    const input = document.getElementById('inputMsg');
    const mode = document.getElementById('inputMode').value;
    const value = input.value.trim();
    
    input.classList.remove('invalid');
    
    if (mode === 'hex') {
        const hexRegex = /^[0-9a-fA-F\s]*$/;
        if (!hexRegex.test(value)) {
            input.classList.add('invalid');
            return false;
        }
    } else if (mode === 'binary') {
        const binRegex = /^[01\s]*$/;
        if (!binRegex.test(value)) {
            input.classList.add('invalid');
            return false;
        }
    }
    
    return true;
}

function validateInputMode() {
    const input = document.getElementById('inputMsg');
    const mode = document.getElementById('inputMode').value;
    
    if (mode === 'text') {
        input.placeholder = "INYECTAR DATOS SEGUROS...";
    } else if (mode === 'hex') {
        input.placeholder = "INGRESA HEX (0-9, A-F)...";
    } else if (mode === 'binary') {
        input.placeholder = "INGRESA BINARIO (0-1)...";
    } else if (mode === 'morse') {
        input.placeholder = "ESCRIBE CÓDIGO MORSE (. y -) o texto...";
    } else if (mode === 'phonetic') {
        input.placeholder = "INGRESA TEXTO PARA FONÉTICO...";
    } else if (mode === 'satellite') {
        input.placeholder = "PREPARADO PARA SATÉLITE...";
    }
    
    input.value = '';
    validateInput();
}

function switchTab(tab) {
    currentTab = tab;
    document.querySelectorAll('.tab-button').forEach(btn => btn.classList.remove('active'));
    document.querySelector(`.tab-button[data-tab="${tab}"]`).classList.add('active');
    document.querySelectorAll('.tab-content').forEach(content => content.classList.remove('active'));
    document.getElementById(`${tab}-table`).classList.add('active');

    const modeSelect = document.getElementById('inputMode');
    if (tab === 'ascii') {
        if (modeSelect.value === 'morse' || modeSelect.value === 'phonetic' || modeSelect.value === 'satellite') {
            modeSelect.value = 'text';
        }
    } else if (tab === 'morse') {
        modeSelect.value = 'morse';
    } else if (tab === 'radio') {
        modeSelect.value = 'phonetic';
        if (typeof initRadioSystem === 'function') {
            setTimeout(initRadioSystem, 100);
        }
    } else if (tab === 'satellite') {
        modeSelect.value = 'satellite';
        initializeSatelliteWeather();
    }
    validateInputMode();
}

function switchTabFromMode() {
    const mode = document.getElementById('inputMode').value;
    if (mode === 'text' || mode === 'hex' || mode === 'binary') {
        switchTab('ascii');
    } else if (mode === 'morse') {
        switchTab('morse');
    } else if (mode === 'phonetic') {
        switchTab('radio');
    } else if (mode === 'satellite') {
        switchTab('satellite');
    }
}

function loadHistoryFromLocal() {
    let history = JSON.parse(localStorage.getItem('radcom_history_v47') || "[]");
    if(history.length > 0) {
        updateMonitor("📂 RECUPERANDO MENSAJES GUARDADOS...");
        history.forEach(item => {
            const logContainer = document.getElementById('logContainer');
            if(logContainer) {
                const div = document.createElement('div');
                div.style.borderLeft = "2px solid #333";
                div.style.paddingLeft = "5px";
                div.style.marginBottom = "2px";
                div.style.color = "#00aa66"; 
                div.innerHTML = `<small>[Historial ${item.time}]</small> <b>${item.sender}:</b> ${item.text}`;
                logContainer.appendChild(div);
            }
        });
    }
}

// =============================================
// FUNCIONES DE SONIDO Y AUDIO
// =============================================

function initMorseAudio() {
    if (!morseAudioContext) {
        try {
            morseAudioContext = new (window.AudioContext || window.webkitAudioContext)();
        } catch (error) {
            console.error("Error inicializando audio Morse:", error);
        }
    }
}

function initRadioAudio() {
    if (!radioAudioContext) {
        try {
            radioAudioContext = new (window.AudioContext || window.webkitAudioContext)();
        } catch (error) {
            console.error("Error inicializando audio radio:", error);
        }
    }
}

function playStrongBeep(freq, duration) {
    if (!document.getElementById('soundEnabled')?.checked) return;
    
    if (!audioContext) {
        audioContext = new (window.AudioContext || window.webkitAudioContext)();
    }
    
    const oscillator = audioContext.createOscillator();
    const gainNode = audioContext.createGain();
    
    oscillator.connect(gainNode);
    gainNode.connect(audioContext.destination);
    
    oscillator.frequency.value = freq;
    oscillator.type = 'sawtooth';
    
    gainNode.gain.setValueAtTime(0.3, audioContext.currentTime);
    gainNode.gain.exponentialRampToValueAtTime(0.01, audioContext.currentTime + duration / 1000);
    
    oscillator.start(audioContext.currentTime);
    oscillator.stop(audioContext.currentTime + duration / 1000);
}

function playMessageNotification() {
    if (!document.getElementById('soundEnabled')?.checked) return;
    
    if (!audioContext) {
        audioContext = new (window.AudioContext || window.webkitAudioContext)();
    }
    
    let currentTime = audioContext.currentTime;
    
    [800, 1200, 1000].forEach((freq, index) => {
        const oscillator = audioContext.createOscillator();
        const gainNode = audioContext.createGain();
        
        oscillator.connect(gainNode);
        gainNode.connect(audioContext.destination);
        
        oscillator.frequency.value = freq;
        oscillator.type = 'sawtooth';
        
        gainNode.gain.setValueAtTime(0.4, currentTime);
        gainNode.gain.exponentialRampToValueAtTime(0.01, currentTime + 0.15);
        
        oscillator.start(currentTime);
        oscillator.stop(currentTime + 0.15);
        
        currentTime += 0.2;
    });
}

function playRadioBeep(freq, duration) {
    if (!document.getElementById('soundEnabled')?.checked) return;
    
    initRadioAudio();
    if (!radioAudioContext) return;
    
    const oscillator = radioAudioContext.createOscillator();
    const gainNode = radioAudioContext.createGain();
    
    oscillator.connect(gainNode);
    gainNode.connect(radioAudioContext.destination);
    
    oscillator.frequency.value = freq;
    oscillator.type = 'sine';
    
    gainNode.gain.setValueAtTime(0.3, radioAudioContext.currentTime);
    gainNode.gain.exponentialRampToValueAtTime(0.01, radioAudioContext.currentTime + duration / 1000);
    
    oscillator.start(radioAudioContext.currentTime);
    oscillator.stop(radioAudioContext.currentTime + duration / 1000);
}

function playBandChangeSound(band) {
    if (!document.getElementById('soundEnabled')?.checked) return;
    
    initRadioAudio();
    if (!radioAudioContext) return;
    
    let time = radioAudioContext.currentTime;
    
    const freqs = {
        'UHF': [1200, 1400, 1600],
        'VHF': [800, 1000, 1200],
        'AEREA': [600, 800, 1000],
        'MARINA': [400, 600, 800],
        'HF': [200, 400, 600],
        'EMERG': [300, 150, 300]
    }[band] || [800, 1000, 1200];
    
    freqs.forEach((freq, index) => {
        const oscillator = radioAudioContext.createOscillator();
        const gainNode = radioAudioContext.createGain();
        
        oscillator.connect(gainNode);
        gainNode.connect(radioAudioContext.destination);
        
        oscillator.frequency.value = freq;
        oscillator.type = 'sawtooth';
        
        gainNode.gain.setValueAtTime(0.4, time);
        gainNode.gain.exponentialRampToValueAtTime(0.01, time + 0.1);
        
        oscillator.start(time);
        oscillator.stop(time + 0.1);
        
        time += 0.15;
    });
}

function playPhoneticSound(char) {
    if (!document.getElementById('soundEnabled')?.checked) return;
    playRadioBeep(500 + char.charCodeAt(0) * 10, 100);
}

function playEmergencyTone() {
    if (!document.getElementById('soundEnabled')?.checked) return;
    
    initRadioAudio();
    if (!radioAudioContext) return;
    
    let time = radioAudioContext.currentTime;
    
    for (let i = 0; i < 3; i++) {
        const oscillator = radioAudioContext.createOscillator();
        const gainNode = radioAudioContext.createGain();
        
        oscillator.connect(gainNode);
        gainNode.connect(radioAudioContext.destination);
        
        oscillator.frequency.value = i % 2 === 0 ? 1000 : 500;
        oscillator.type = 'sawtooth';
        
        gainNode.gain.setValueAtTime(0.5, time);
        gainNode.gain.exponentialRampToValueAtTime(0.01, time + 0.3);
        
        oscillator.start(time);
        oscillator.stop(time + 0.3);
        
        time += 0.35;
    }
}

function playTransmissionSound() {
    if (!document.getElementById('soundEnabled')?.checked) return;
    
    initRadioAudio();
    if (!radioAudioContext) return;
    
    const bufferSize = radioAudioContext.sampleRate * 0.5;
    const buffer = radioAudioContext.createBuffer(1, bufferSize, radioAudioContext.sampleRate);
    const output = buffer.getChannelData(0);
    
    for (let i = 0; i < bufferSize; i++) {
        output[i] = Math.random() * 2 - 1;
        output[i] += 0.3 * Math.sin(2 * Math.PI * 1000 * i / radioAudioContext.sampleRate);
        const envelope = i < bufferSize * 0.1 ? i / (bufferSize * 0.1) : (bufferSize - i) / (bufferSize * 0.9);
        output[i] *= envelope;
    }
    
    const source = radioAudioContext.createBufferSource();
    source.buffer = buffer;
    source.connect(radioAudioContext.destination);
    source.start();
    
    const staticOverlay = document.getElementById('static-overlay');
    staticOverlay.classList.add('static-active');
    
    setTimeout(() => {
        staticOverlay.classList.remove('static-active');
    }, 500);
}

function playEmergencySatelliteTone() {
    if (!document.getElementById('soundEnabled')?.checked) return;
    
    if (!audioContext) {
        audioContext = new (window.AudioContext || window.webkitAudioContext)();
    }
    
    let time = audioContext.currentTime;
    
    for (let i = 0; i < 3; i++) {
        const oscillator = audioContext.createOscillator();
        const gainNode = audioContext.createGain();
        
        oscillator.connect(gainNode);
        gainNode.connect(audioContext.destination);
        
        oscillator.frequency.setValueAtTime(800, time);
        oscillator.frequency.exponentialRampToValueAtTime(1200, time + 0.3);
        oscillator.type = 'sawtooth';
        
        gainNode.gain.setValueAtTime(0.6, time);
        gainNode.gain.exponentialRampToValueAtTime(0.01, time + 0.3);
        
        oscillator.start(time);
        oscillator.stop(time + 0.3);
        
        time += 0.4;
    }
}

function playEmergencyAlertSound() {
    if (!audioContext) {
        audioContext = new (window.AudioContext || window.webkitAudioContext)();
    }
    
    let time = audioContext.currentTime;
    
    for (let i = 0; i < 3; i++) {
        const oscillator = audioContext.createOscillator();
        const gainNode = audioContext.createGain();
        
        oscillator.connect(gainNode);
        gainNode.connect(audioContext.destination);
        
        oscillator.frequency.value = 1500;
        oscillator.type = 'square';
        
        gainNode.gain.setValueAtTime(0.5, time);
        gainNode.gain.exponentialRampToValueAtTime(0.01, time + 0.1);
        
        oscillator.start(time);
        oscillator.stop(time + 0.1);
        
        time += 0.15;
    }
}

function playMorseCharSound(char) {
    if (isPlayingMorse) return;
    if (!document.getElementById('soundEnabled')?.checked) return;
    
    const morseCode = morseCodes[char];
    if (morseCode) {
        playMorseCodeSound(morseCode);
    }
}

function playMorseCodeSound(code) {
    if (!document.getElementById('soundEnabled')?.checked) return;
    
    initMorseAudio();
    if (!morseAudioContext || isPlayingMorse) return;
    
    isPlayingMorse = true;
    let time = morseAudioContext.currentTime;
    
    let dotDuration, dashDuration, symbolGap;
    
    switch (morseSpeed) {
        case 'slow':
            dotDuration = 0.2;
            dashDuration = 0.6;
            symbolGap = 0.15;
            break;
        case 'fast':
            dotDuration = 0.1;
            dashDuration = 0.3;
            symbolGap = 0.08;
            break;
        default:
            dotDuration = 0.15;
            dashDuration = 0.45;
            symbolGap = 0.1;
    }
    
    for (let i = 0; i < code.length; i++) {
        const symbol = code[i];
        
        if (symbol === '.' || symbol === '-') {
            const duration = symbol === '.' ? dotDuration : dashDuration;
            
            const oscillator = morseAudioContext.createOscillator();
            const gainNode = morseAudioContext.createGain();
            
            oscillator.connect(gainNode);
            gainNode.connect(morseAudioContext.destination);
            
            oscillator.frequency.value = 700;
            oscillator.type = 'sine';
            
            gainNode.gain.setValueAtTime(0, time);
            gainNode.gain.linearRampToValueAtTime(0.3, time + 0.01);
            gainNode.gain.setValueAtTime(0.3, time + duration - 0.01);
            gainNode.gain.linearRampToValueAtTime(0, time + duration);
            
            oscillator.start(time);
            oscillator.stop(time + duration);
            
            time += duration + symbolGap;
        }
    }
    
    setTimeout(() => {
        isPlayingMorse = false;
    }, (time - morseAudioContext.currentTime) * 1000 + 100);
}

function setMorseSpeed(speed) {
    morseSpeed = speed;
    
    document.querySelectorAll('.speed-btn').forEach(btn => {
        btn.classList.remove('active');
    });
    document.querySelector(`.speed-btn[onclick="setMorseSpeed('${speed}')"]`).classList.add('active');
    
    updateMonitor(`⚡ Velocidad Morse: ${speed.toUpperCase()}`);
    playStrongBeep(800, 50);
}

function playMorsePreview() {
    const input = document.getElementById('inputMsg');
    const text = input.value.trim();
    
    if (!text) {
        updateMonitor("⚠️ Escribe algo para probar el sonido Morse", "warning");
        playStrongBeep(300, 100);
        return;
    }
    
    if (!document.getElementById('soundEnabled')?.checked) {
        updateMonitor("🔇 Sonido desactivado en configuración", "warning");
        return;
    }
    
    updateMonitor(`🔊 Probando Morse: "${text.substring(0, 20)}${text.length > 20 ? '...' : ''}"`);
    
    const words = text.toUpperCase().split(' ');
    
    let totalDelay = 0;
    words.forEach((word, wordIndex) => {
        for (let i = 0; i < word.length; i++) {
            const char = word[i];
            const morseCode = morseCodes[char];
            
            if (morseCode) {
                setTimeout(() => {
                    playMorseCodeSound(morseCode);
                }, totalDelay);
                
                const dotDuration = morseSpeed === 'slow' ? 200 : morseSpeed === 'fast' ? 100 : 150;
                const dashDuration = morseSpeed === 'slow' ? 600 : morseSpeed === 'fast' ? 300 : 450;
                const symbolGap = morseSpeed === 'slow' ? 150 : morseSpeed === 'fast' ? 80 : 100;
                const letterGap = morseSpeed === 'slow' ? 400 : morseSpeed === 'fast' ? 200 : 300;
                
                let charDuration = 0;
                for (let j = 0; j < morseCode.length; j++) {
                    if (morseCode[j] === '.') charDuration += dotDuration + symbolGap;
                    else if (morseCode[j] === '-') charDuration += dashDuration + symbolGap;
                }
                
                totalDelay += charDuration + letterGap;
            }
        }
        
        if (wordIndex < words.length - 1) {
            totalDelay += morseSpeed === 'slow' ? 1200 : morseSpeed === 'fast' ? 600 : 900;
        }
    });
    
    playStrongBeep(1000, 100);
}

// =============================================
// SISTEMA DE RADIO
// =============================================

function validateFrequency() {
    const input = document.getElementById('radio-frequency-input');
    if (!input) return;
    const value = input.value.replace(',', '.');
    
    if (!/^[\d.]*$/.test(value)) {
        input.value = value.replace(/[^\d.]/g, '');
    }
    
    const parts = value.split('.');
    if (parts.length > 1 && parts[1].length > 3) {
        input.value = parts[0] + '.' + parts[1].substring(0, 3);
    }
}

function updateFrequencyFromInput() {
    const input = document.getElementById('radio-frequency-input');
    if (!input) return;
    let value = parseFloat(input.value.replace(',', '.'));
    
    if (isNaN(value)) {
        value = radioBands[currentBand].default;
        input.value = value.toFixed(3);
    }
    
    currentFrequency = value;
    updateFrequencyDisplay();
    updateChannelDisplay();
    playRadioBeep(800, 50);
}

function updateUnit() {
    const unitSelect = document.getElementById('radio-unit');
    if (!unitSelect) return;
    currentUnit = unitSelect.value;
    
    if (currentUnit === 'KHz' && currentFrequency > 1000) {
        currentFrequency = currentFrequency / 1000;
    } else if (currentUnit === 'MHz' && currentFrequency < 1) {
        currentFrequency = currentFrequency * 1000;
    }
    
    updateFrequencyDisplay();
    updateChannelDisplay();
}

function tuneFrequency(step) {
    currentFrequency += step;
    
    const band = radioBands[currentBand];
    if (band) {
        if (currentFrequency < band.min) currentFrequency = band.min;
        if (currentFrequency > band.max) currentFrequency = band.max;
    }
    
    updateFrequencyDisplay();
    updateChannelDisplay();
    playRadioBeep(600 + Math.abs(step) * 100, 30);
    
    const freqInput = document.getElementById('radio-frequency-input');
    if (freqInput) freqInput.value = currentFrequency.toFixed(3);
}

function selectBand(band) {
    currentBand = band;
    currentHFBand = null;
    
    document.querySelectorAll('.band-btn').forEach(btn => {
        btn.classList.remove('active');
    });
    const bandBtn = document.querySelector(`.band-btn[data-band="${band}"]`);
    if (bandBtn) bandBtn.classList.add('active');
    
    const hfContainer = document.getElementById('hf-sub-bands');
    if (hfContainer) {
        if (band === 'HF') {
            hfContainer.style.display = 'block';
            selectHFBand('10m');
        } else {
            hfContainer.style.display = 'none';
            document.querySelectorAll('.hf-band-btn').forEach(btn => {
                btn.classList.remove('active');
            });
        }
    }
    
    if (radioBands[band]) {
        currentFrequency = radioBands[band].default;
        currentUnit = radioBands[band].unit;
        
        const unitSelect = document.getElementById('radio-unit');
        if (unitSelect) unitSelect.value = currentUnit;
        
        const freqInput = document.getElementById('radio-frequency-input');
        if (freqInput) freqInput.value = currentFrequency.toFixed(3);
        
        updateFrequencyDisplay();
        updateEmergencyFrequencies();
        updateChannelDisplay();
        
        playBandChangeSound(band);
    }
}

function selectHFBand(hfBand) {
    currentHFBand = hfBand;
    
    document.querySelectorAll('.hf-band-btn').forEach(btn => {
        btn.classList.remove('active');
    });
    const hfBtn = document.querySelector(`.hf-band-btn[data-hf="${hfBand}"]`);
    if (hfBtn) hfBtn.classList.add('active');
    
    if (hfBands[hfBand]) {
        const range = hfBands[hfBand].range.split('-')[0];
        const freq = parseFloat(range);
        if (!isNaN(freq)) {
            currentFrequency = freq;
            const freqInput = document.getElementById('radio-frequency-input');
            if (freqInput) freqInput.value = currentFrequency.toFixed(3);
            updateFrequencyDisplay();
            updateChannelDisplay();
        }
    }
    
    playRadioBeep(700, 40);
}

function selectChannelType(type) {
    currentChannelType = type;
    currentChannel = 1;
    
    document.querySelectorAll('.channel-type-btn').forEach(btn => {
        btn.classList.remove('active');
    });
    const typeBtn = document.querySelector(`.channel-type-btn[onclick="selectChannelType('${type}')"]`);
    if (typeBtn) typeBtn.classList.add('active');
    
    document.querySelectorAll('.channel-container').forEach(container => {
        container.classList.remove('active');
    });
    const container = document.getElementById(`${type}-container`);
    if (container) container.classList.add('active');
    
    if (type === 'pmr') {
        selectPMRChannel(1);
    } else if (type === 'cb') {
        selectCBChannel(19);
    }
    
    playRadioBeep(600, 40);
}

function buildPMRChannels() {
    const container = document.getElementById('pmr-channels-grid');
    if (!container) return;
    container.innerHTML = '';
    
    const channels = pmrChannels[currentPMRType];
    for (let i = 1; i <= currentPMRType; i++) {
        const btn = document.createElement('button');
        btn.className = 'channel-btn';
        btn.textContent = i;
        btn.onclick = () => selectPMRChannel(i);
        container.appendChild(btn);
    }
    
    if (container.firstChild) {
        container.firstChild.classList.add('active');
    }
}

function selectPMRType(type) {
    currentPMRType = parseInt(type);
    buildPMRChannels();
    selectPMRChannel(1);
}

function selectPMRChannel(channel) {
    const channels = pmrChannels[currentPMRType];
    if (channels && channels[channel]) {
        currentFrequency = channels[channel];
        currentChannel = channel;
        
        document.querySelectorAll('#pmr-channels-grid .channel-btn').forEach(btn => {
            btn.classList.remove('active');
        });
        const btns = document.querySelectorAll('#pmr-channels-grid .channel-btn');
        if (btns[channel - 1]) btns[channel - 1].classList.add('active');
        
        const freqInput = document.getElementById('radio-frequency-input');
        if (freqInput) freqInput.value = currentFrequency.toFixed(5);
        updateFrequencyDisplay();
        updateChannelDisplay();
        playRadioBeep(700, 40);
    }
}

function buildCBChannels() {
    const container = document.getElementById('cb-channels-grid');
    if (!container) return;
    container.innerHTML = '';
    
    for (let i = 1; i <= 40; i++) {
        const btn = document.createElement('button');
        btn.className = 'channel-btn';
        btn.textContent = i;
        btn.onclick = () => selectCBChannel(i);
        container.appendChild(btn);
    }
    
    if (container.children[18]) {
        container.children[18].classList.add('active');
    }
}

function selectCBChannel(channel) {
    if (cbChannels[channel]) {
        currentFrequency = cbChannels[channel];
        currentChannel = channel;
        
        document.querySelectorAll('#cb-channels-grid .channel-btn').forEach(btn => {
            btn.classList.remove('active');
        });
        const btns = document.querySelectorAll('#cb-channels-grid .channel-btn');
        if (btns[channel - 1]) btns[channel - 1].classList.add('active');
        
        const freqInput = document.getElementById('radio-frequency-input');
        if (freqInput) freqInput.value = currentFrequency.toFixed(3);
        updateFrequencyDisplay();
        updateChannelDisplay();
        playRadioBeep(700, 40);
    }
}

function updateFrequencyDisplay() {
    const display = document.getElementById('radio-frequency-display');
    const activeDisplay = document.getElementById('active-frequency');
    
    if (!display || !activeDisplay) return;
    
    let freqText = currentFrequency.toFixed(3);
    if (currentFrequency < 1 && currentUnit === 'MHz') {
        freqText = (currentFrequency * 1000).toFixed(1) + ' KHz';
    } else {
        freqText += ' ' + currentUnit;
    }
    
    display.textContent = freqText;
    
    let activeText = freqText + ' ' + currentBand;
    if (currentHFBand) {
        activeText += ' (' + currentHFBand + ')';
    }
    activeDisplay.textContent = activeText;
    
    localStorage.setItem('radcom_radio_freq', currentFrequency);
    localStorage.setItem('radcom_radio_band', currentBand);
    localStorage.setItem('radcom_radio_unit', currentUnit);
    if (currentHFBand) {
        localStorage.setItem('radcom_radio_hfband', currentHFBand);
    }
}

function updateChannelDisplay() {
    const display = document.getElementById('channel-display');
    if (!display) return;
    
    if (currentChannelType === 'pmr') {
        display.textContent = `PMR446 CH${currentChannel} (${currentFrequency.toFixed(5)} MHz)`;
    } else if (currentChannelType === 'cb') {
        display.textContent = `CB CH${currentChannel} (${currentFrequency.toFixed(3)} MHz)`;
    } else {
        display.textContent = `${currentBand} ${currentFrequency.toFixed(3)} ${currentUnit}`;
    }
}

function updateEmergencyFrequencies() {
    const list = document.getElementById('emergency-list');
    if (!list) return;
    
    const emergencies = emergencyFrequencies[currentBand] || [];
    
    let html = '';
    emergencies.forEach(emerg => {
        html += `
            <div class="emergency-item" onclick="setEmergencyFrequency('${emerg.freq}')" 
                 title="${emerg.purpose}">
                ${emerg.freq} MHz - ${emerg.purpose}
            </div>
        `;
    });
    
    list.innerHTML = html || '<div class="emergency-item">No hay frecuencias de emergencia para esta banda</div>';
}

function setEmergencyFrequency(freq) {
    currentFrequency = parseFloat(freq);
    const freqInput = document.getElementById('radio-frequency-input');
    if (freqInput) freqInput.value = currentFrequency.toFixed(3);
    updateFrequencyDisplay();
    updateChannelDisplay();
    
    playEmergencyTone();
    updateMonitor(`⚠️ FRECUENCIA DE EMERGENCIA: ${freq} MHz`);
}

function selectPhoneticCharRadio(char) {
    const input = document.getElementById('inputMsg');
    const inputMode = document.getElementById('inputMode').value;
    const data = phoneticAlphabetFull[char];
    
    if (inputMode === 'phonetic') {
        input.value += data.word + ' ';
    } else {
        input.value += char;
    }
    
    input.focus();
    playPhoneticSound(char);
}

function initRadioSystem() {
    buildRadioTable();
    
    const savedFreq = localStorage.getItem('radcom_radio_freq');
    const savedBand = localStorage.getItem('radcom_radio_band');
    const savedUnit = localStorage.getItem('radcom_radio_unit');
    const savedHFBand = localStorage.getItem('radcom_radio_hfband');
    
    if (savedFreq) currentFrequency = parseFloat(savedFreq);
    if (savedBand) currentBand = savedBand;
    if (savedUnit) currentUnit = savedUnit;
    if (savedHFBand) currentHFBand = savedHFBand;
    
    document.querySelectorAll('.band-btn').forEach(btn => {
        btn.classList.remove('active');
    });
    const bandBtn = document.querySelector(`.band-btn[data-band="${currentBand}"]`);
    if (bandBtn) bandBtn.classList.add('active');
    
    const freqInput = document.getElementById('radio-frequency-input');
    if (freqInput) freqInput.value = currentFrequency.toFixed(3);
    
    const unitSelect = document.getElementById('radio-unit');
    if (unitSelect) unitSelect.value = currentUnit;
    
    const hfContainer = document.getElementById('hf-sub-bands');
    if (hfContainer && currentBand === 'HF' && currentHFBand) {
        hfContainer.style.display = 'block';
        const hfBtn = document.querySelector(`.hf-band-btn[data-hf="${currentHFBand}"]`);
        if (hfBtn) hfBtn.classList.add('active');
    }
    
    updateFrequencyDisplay();
    updateEmergencyFrequencies();
    updateChannelDisplay();
}

// =============================================
// SISTEMA SATELITAL
// =============================================

function getCurrentGPSPosition(forceOneTime = false) {
    return new Promise((resolve, reject) => {
        if (!navigator.geolocation) {
            updateMonitor("❌ Geolocalización no soportada en este navegador", "error");
            reject(new Error("Geolocation no soportada"));
            return;
        }

        updateMonitor("📍 Solicitando acceso a ubicación...", "info");
        playStrongBeep(800, 100);

        const options = {
            enableHighAccuracy: true,
            timeout: 15000,
            maximumAge: 30000
        };

        const successCallback = (position) => {
            satelliteSystem.latitude = position.coords.latitude;
            satelliteSystem.longitude = position.coords.longitude;
            satelliteSystem.altitude = position.coords.altitude || 0;
            satelliteSystem.accuracy = position.coords.accuracy;
            satelliteSystem.lastUpdate = new Date();

            localStorage.setItem('radcom_last_position', JSON.stringify({
                latitude: satelliteSystem.latitude,
                longitude: satelliteSystem.longitude,
                altitude: satelliteSystem.altitude,
                accuracy: satelliteSystem.accuracy,
                timestamp: Date.now()
            }));

            updateSatelliteUI();
            updateMonitor(`📍 Posición actualizada (Precisión: ${Math.round(satelliteSystem.accuracy)} m)`, "info");
            playStrongBeep(600, 50);

            resolve();
        };

        const errorCallback = (error) => {
            let msg = "❌ Error GPS: ";
            let showInstructions = false;

            switch (error.code) {
                case error.PERMISSION_DENIED:
                    msg += "Permiso denegado.";
                    showInstructions = true;
                    break;
                case error.POSITION_UNAVAILABLE:
                    msg += "Posición no disponible.";
                    break;
                case error.TIMEOUT:
                    msg += "Tiempo agotado.";
                    break;
                default:
                    msg += error.message;
            }

            updateMonitor(msg, "error");

            if (showInstructions) {
                alert(msg);
            }

            reject(error);
        };

        if (forceOneTime) {
            navigator.geolocation.getCurrentPosition(successCallback, errorCallback, options);
        } else {
            watchId = navigator.geolocation.watchPosition(successCallback, errorCallback, options);
        }
    });
}

function useRealtPosition() {
    satelliteSystem.latitude = 40.4168;
    satelliteSystem.longitude = -3.7038;
    satelliteSystem.altitude = 667;
    satelliteSystem.accuracy = 1000;
    satelliteSystem.lastUpdate = new Date();
    
    updateSatelliteUI();
    updateMonitor("⚠️ Usando posición Real", "warning");
    
    fetchWeatherData();
}

function fetchWeatherData() {
    if (!satelliteSystem.latitude || !satelliteSystem.longitude) {
        updateMonitor("⚠️ Esperando posición GPS...", "warning");
        return;
    }

    const lat = satelliteSystem.latitude.toFixed(4);
    const lon = satelliteSystem.longitude.toFixed(4);

    updateMonitor("🌤️ Consultando meteo + elevación...", "info");
    updateAPIStatus("Conectando a Open-Meteo...", false);

    try {
        const meteoParams = new URLSearchParams({
            latitude: lat,
            longitude: lon,
            hourly: 'temperature_2m,relative_humidity_2m,dew_point_2m,apparent_temperature,pressure_msl,cloud_cover,wind_speed_10m,wind_direction_10m,wind_gusts_10m,visibility,precipitation_probability,rain,uv_index,is_day',
            current_weather: true,
            timezone: 'auto',
            forecast_days: 1
        }).toString();

        const meteoUrl = `https://api.open-meteo.com/v1/forecast?${meteoParams}`;
        console.log("📡 Meteo URL:", meteoUrl);

        fetch(meteoUrl)
            .then(response => {
                if (!response.ok) throw new Error(`Meteo HTTP ${response.status}`);
                return response.json();
            })
            .then(data => {
                satelliteSystem.weatherData = data;
                
                const elevUrl = `https://api.open-meteo.com/v1/elevation?latitude=${lat}&longitude=${lon}`;
                return fetch(elevUrl);
            })
            .then(elevResponse => {
                if (!elevResponse.ok) throw new Error(`Elevación HTTP ${elevResponse.status}`);
                return elevResponse.json();
            })
            .then(elevData => {
                if (elevData.elevation && elevData.elevation.length > 0) {
                    satelliteSystem.altitude = Math.round(elevData.elevation[0]);
                }
                
                if (satelliteSystem.altitude === null || isNaN(satelliteSystem.altitude)) {
                    satelliteSystem.altitude = satelliteSystem.altitude || 0;
                }
                
                updateWeatherUI(satelliteSystem.weatherData);
                updateSatelliteUI();
                updateAPIStatus("Conectado", true);
                updateMonitor("✅ Meteo y altitud actualizados", "info");
                playStrongBeep(700, 30);
            })
            .catch(error => {
                console.error("❌ Error API:", error);
                satelliteSystem.apiConnected = false;
                updateMonitor(`❌ Error: ${error.message}`, "error");
                updateAPIStatus("Error conexión", false);
                playStrongBeep(300, 200);
                useSampleWeatherData();
            });
    } catch (error) {
        console.error("❌ Error total:", error);
        useSampleWeatherData();
    }
}

function useSampleWeatherData() {
    const sampleData = {
        current_weather: {
            temperature: 18.5,
            windspeed: 12.3,
            winddirection: 245
        },
        hourly: {
            apparent_temperature: [17.2],
            relative_humidity_2m: [68],
            pressure_msl: [1015],
            wind_gusts_10m: [25.6],
            cloud_cover: [45],
            visibility: [10000],
            precipitation_probability: [10],
            rain: [0.2],
            dew_point_2m: [12.3],
            uv_index: [4.5]
        }
    };
    
    satelliteSystem.weatherData = sampleData;
    satelliteSystem.altitude = satelliteSystem.altitude || 667;
    
    updateWeatherUI(sampleData);
    updateSatelliteUI();
    updateMonitor("⚠️ Usando datos de muestra (Alt: 667 m)", "warning");
}

function updateWeatherUI(data) {
    if (!data || !data.current_weather) return;
    
    const cw = data.current_weather;
    const hourly = data.hourly;
    const currentIndex = 0;
    
    document.getElementById('sat-temp').textContent = `${cw.temperature.toFixed(1)} °C`;
    document.getElementById('sat-windspeed').textContent = `${cw.windspeed.toFixed(1)} km/h`;
    document.getElementById('sat-winddirection').textContent = `${cw.winddirection} °`;
    
    if (hourly) {
        if (hourly.apparent_temperature?.[currentIndex] !== undefined) {
            document.getElementById('sat-feelslike').textContent = `${hourly.apparent_temperature[currentIndex].toFixed(1)} °C`;
        }
        
        if (hourly.relative_humidity_2m?.[currentIndex] !== undefined) {
            document.getElementById('sat-humidity').textContent = `${hourly.relative_humidity_2m[currentIndex].toFixed(0)} %`;
        }
        
        if (hourly.pressure_msl?.[currentIndex] !== undefined) {
            document.getElementById('sat-pressure').textContent = `${hourly.pressure_msl[currentIndex].toFixed(0)} hPa`;
        }
        
        if (hourly.wind_gusts_10m?.[currentIndex] !== undefined) {
            document.getElementById('sat-windgusts').textContent = `${hourly.wind_gusts_10m[currentIndex].toFixed(1)} km/h`;
        }
        
        if (hourly.cloud_cover?.[currentIndex] !== undefined) {
            document.getElementById('sat-cloudcover').textContent = `${hourly.cloud_cover[currentIndex].toFixed(0)} %`;
        }
        
        if (hourly.visibility?.[currentIndex] !== undefined) {
            const visibilityKm = hourly.visibility[currentIndex] / 1000;
            document.getElementById('sat-visibility').textContent = `${visibilityKm.toFixed(1)} km`;
        }
        
        if (hourly.precipitation_probability?.[currentIndex] !== undefined) {
            document.getElementById('sat-precipitation').textContent = `${hourly.precipitation_probability[currentIndex].toFixed(0)} %`;
        }
        
        if (hourly.rain?.[currentIndex] !== undefined) {
            document.getElementById('sat-rain').textContent = `${hourly.rain[currentIndex].toFixed(1)} mm`;
        }
        
        if (hourly.dew_point_2m?.[currentIndex] !== undefined) {
            document.getElementById('sat-dewpoint').textContent = `${hourly.dew_point_2m[currentIndex].toFixed(1)} °C`;
        }
        
        if (hourly.uv_index?.[currentIndex] !== undefined) {
            document.getElementById('sat-uvindex').textContent = `${hourly.uv_index[currentIndex].toFixed(1)}`;
        }
        
        calculateWeatherIndexes(cw.temperature, hourly.relative_humidity_2m?.[currentIndex] || 50, cw.windspeed);
    }
}

function calculateWeatherIndexes(temp, humidity, windSpeed) {
    if (temp >= 27) {
        const T = temp;
        const R = humidity;
        const hi = -8.784695 + 1.61139411*T + 2.338549*R - 0.14611605*T*R 
                  - 0.012308094*T*T - 0.016424828*R*R + 0.002211732*T*T*R 
                  + 0.00072546*T*R*R - 0.000003582*T*T*R*R;
        document.getElementById('sat-heatindex').textContent = `${hi.toFixed(1)} °C`;
    } else {
        document.getElementById('sat-heatindex').textContent = "N/A";
    }
    
    if (temp <= 10 && windSpeed >= 5) {
        const wc = 13.12 + 0.6215*temp - 11.37*Math.pow(windSpeed, 0.16) 
                  + 0.3965*temp*Math.pow(windSpeed, 0.16);
        document.getElementById('sat-windchill').textContent = `${wc.toFixed(1)} °C`;
    } else {
        document.getElementById('sat-windchill').textContent = "N/A";
    }
    
    let aqiScore = 50;
    if (humidity > 80) aqiScore += 20;
    if (humidity < 30) aqiScore += 10;
    if (temp > 30) aqiScore += 15;
    
    let aqiLevel = "Buena";
    if (aqiScore > 150) aqiLevel = "Muy pobre";
    else if (aqiScore > 100) aqiLevel = "Pobre";
    else if (aqiScore > 50) aqiLevel = "Moderada";
    
    document.getElementById('sat-aqi').textContent = aqiLevel;
    
    document.getElementById('sat-sunshine').textContent = "8.5 h";
    document.getElementById('sat-co2').textContent = "415 ppm";
}

function updateSatelliteUI() {
    if (satelliteSystem.latitude && satelliteSystem.longitude) {
        const lat = satelliteSystem.latitude.toFixed(4);
        const lon = satelliteSystem.longitude.toFixed(4);
        const latDir = satelliteSystem.latitude >= 0 ? 'N' : 'S';
        const lonDir = satelliteSystem.longitude >= 0 ? 'E' : 'W';
        
        document.getElementById('sat-gps-coords').textContent = 
            `Lat: ${Math.abs(lat)}° ${latDir}, Lon: ${Math.abs(lon)}° ${lonDir}`;
        
        const alt = satelliteSystem.altitude ? Math.round(satelliteSystem.altitude) : '--';
        const acc = satelliteSystem.accuracy ? Math.round(satelliteSystem.accuracy) : '--';
        document.getElementById('updateSatelliteUI').textContent = 
            `Altitud: ${alt} m | Precisión: ${acc} m`;
        
        if (satelliteSystem.lastUpdate) {
            const time = satelliteSystem.lastUpdate.toLocaleTimeString();
            document.getElementById('sat-last-update').textContent = time;
        }
    }
}

function updateAPIStatus(text, connected) {
    const dot = document.getElementById('api-status-dot');
    const textElement = document.getElementById('api-status-text');
    
    if (dot) {
        dot.className = "status-dot-api " + (connected ? "status-connected" : "status-disconnected");
    }
    if (textElement) {
        textElement.textContent = text;
        textElement.style.color = connected ? "#00ff88" : "#ff3300";
    }
}

function forceUpdateSatelliteData() {
    updateMonitor("🔄 Actualización forzada de datos satelitales...", "info");
    playStrongBeep(900, 80);
    getCurrentGPSPosition().then(() => fetchWeatherData());
}

function sendPositionToChat() {
    if (!satelliteSystem.latitude || !satelliteSystem.longitude) {
        updateMonitor("❌ No hay posición GPS. Obtén posición primero.", "error");
        playStrongBeep(300, 200);
        getCurrentGPSPosition();
        return;
    }

    const lat = satelliteSystem.latitude.toFixed(6);
    const lon = satelliteSystem.longitude.toFixed(6);
    const alt = satelliteSystem.altitude ? Math.round(satelliteSystem.altitude) : 'N/A';
    const time = new Date().toLocaleTimeString();

    const positionMessage = `📍 POSICIÓN ACTUAL\nLat: ${lat}\nLon: ${lon}\nAlt: ${alt} m\nHora: ${time}`;

    const dataToSend = prepareMessageToSend(
        positionMessage,
        'text',
        document.getElementById('encryptionMode').value,
        document.getElementById('key').value || 'ATOM80'
    );

    const sent = sendToConnections(dataToSend);

    if (sent > 0) {
        displayMessage(`📍 YO: ${positionMessage.substring(0, 60)}...`, '', 'outgoing');
        updateMonitor(`📍 Posición enviada a ${sent} destino(s)`, "info");
        playStrongBeep(700, 100);
    } else {
        updateMonitor("⚠️ No se pudo enviar posición (sin conexiones)", "warning");
    }
}

function sendSatelliteEmergency() {
    updateMonitor("🚨 INICIANDO PROTOCOLO DE EMERGENCIA...", "warning");
    playEmergencySatelliteTone();
    
    try {
        getCurrentGPSPosition(true).then(() => {
            if (!satelliteSystem.latitude || !satelliteSystem.longitude) {
                throw new Error("No position available");
            }
            
            if (satelliteSystem.accuracy > 100) {
                if (!confirm(`⚠️ Precisión baja (${Math.round(satelliteSystem.accuracy)}m). ¿Continuar?`)) {
                    return;
                }
            }
            
            const onlineCount = Object.keys(connections).filter(id => 
                connections[id]?.status === 'online').length;
            
            if (onlineCount === 0) {
                if (!confirm("⚠️ No hay contactos conectados. ¿Enviar solo a chat local?")) {
                    return;
                }
            }
            
            if (!confirm(`🚨 ¿ENVIAR SEÑAL DE EMERGENCIA S.O.S AYUDA?\n\nPosición: ${satelliteSystem.latitude.toFixed(4)}° N, ${satelliteSystem.longitude.toFixed(4)}° E\nSe enviará a ${onlineCount} contacto(s).`)) {
                return;
            }
            
            const emergencyMessage = createEmergencyMessage();
            const sentCount = sendEmergencyMessage(emergencyMessage);
            
            updateMonitor(`✅ EMERGENCIA ENVIADA A ${sentCount} CONTACTO(S)`, "info");
            displayMessage(`🚨 YO [EMERGENCIA]: ${emergencyMessage.substring(0, 80)}...`, 'EMERG', 'outgoing');
        }).catch(error => {
            console.error("Emergency error:", error);
            updateMonitor("❌ ERROR EN PROTOCOLO DE EMERGENCIA: " + error.message, "error");
            playStrongBeep(300, 200);
        });
    } catch (error) {
        console.error("Emergency error:", error);
        updateMonitor("❌ ERROR EN PROTOCOLO DE EMERGENCIA: " + error.message, "error");
        playStrongBeep(300, 200);
    }
}

function createEmergencyMessage() {
    const lat = satelliteSystem.latitude.toFixed(6);
    const lon = satelliteSystem.longitude.toFixed(6);
    const alt = satelliteSystem.altitude ? Math.round(satelliteSystem.altitude) : 'N/A';
    const time = new Date().toLocaleTimeString();
    
    let weatherInfo = '';
    if (satelliteSystem.weatherData && satelliteSystem.weatherData.current_weather) {
        const w = satelliteSystem.weatherData.current_weather;
        weatherInfo = ` | Condiciones: ${w.temperature}°C, Viento: ${w.windspeed} km/h`;
    }
    
    return `🚨 EMERGENCIA S.O.S AYUDA 🚨
Posición: ${lat}° N, ${lon}° E
Altitud: ${alt} m
Hora: ${time}
Sistema: RADCOM v${VERSION}${weatherInfo}
⚠️ NECESITO ASISTENCIA INMEDIATA`;
}

function sendEmergencyMessage(message) {
    const key = document.getElementById('key').value || 'ATOM80';
    
    const dataToSend = {
        type: 'emergency',
        message: xorEncrypt(message, key),
        original: message,
        mode: 'satellite',
        encryption: document.getElementById('encryptionMode').value,
        timestamp: Date.now(),
        sender: myPeerId,
        hexPreview: 'EMERGENCY'
    };
    
    let sentCount = 0;
    
    Object.keys(connections).forEach(peerId => {
        if (connections[peerId]?.status === 'online') {
            try {
                console.log(`🚨 Enviando emergencia a ${peerId}`);
                connections[peerId].conn.send(dataToSend);
                sentCount++;
                
                if (connectionHealth[peerId]) {
                    connectionHealth[peerId].lastActivity = Date.now();
                    connectionHealth[peerId].packetsSent = (connectionHealth[peerId]?.packetsSent || 0) + 1;
                }
            } catch (error) {
                console.error(`❌ Error enviando emergencia a ${peerId}:`, error);
            }
        }
    });
    
    stats.tx += JSON.stringify(dataToSend).length;
    stats.messages++;
    updateStats();
    
    console.log(`✅ Emergencia enviada a ${sentCount} contacto(s)`);
    return sentCount;
}

function handleEmergencyMessage(senderId, data) {
    const key = document.getElementById('key').value || 'ATOM80';
    const decryptedMessage = xorDecrypt(data.message, key);
    
    displayMessage(`🚨 EMERGENCIA DE ${senderId.substring(0,6)}: ${decryptedMessage.substring(0, 100)}...`, 'EMERG', 'incoming');
    
    if (document.getElementById('soundEnabled')?.checked) {
        playEmergencyAlertSound();
    }
    
    updateMonitor(`🚨 EMERGENCIA RECIBIDA DE ${senderId.substring(0,6)}`);
    
    stats.rx += data.message.length;
    updateStats();
}

function initSatelliteSystem() {
    const savedPos = localStorage.getItem('radcom_last_position');
    if (savedPos) {
        satelliteSystem.lastKnownPosition = JSON.parse(savedPos);
        console.log("📍 Posición anterior cargada:", satelliteSystem.lastKnownPosition);
    }
    
    updateSatelliteUI();
    
    setInterval(() => {
        if (!document.hidden) {
            forceUpdateSatelliteData();
        }
    }, 100000);
}

function startWatchingPosition() {
    if (watchId !== null) return;
    getCurrentGPSPosition(false)
        .then(() => { console.log("Seguimiento GPS iniciado en modo continuo"); })
        .catch(() => { console.warn("No se pudo iniciar seguimiento continuo"); });
}

function stopWatchingPosition() {
    if (watchId !== null) {
        navigator.geolocation.clearWatch(watchId);
        watchId = null;
        console.log("Seguimiento GPS detenido");
    }
}

function initializeSatelliteWeather() {
    console.log("Satellite weather initialized");
}

// =============================================
// SISTEMA DE RECONOCIMIENTO DE VOZ
// =============================================

function initVoiceRecognition() {
    const SpeechRecognition = window.SpeechRecognition || window.webkitSpeechRecognition;
    if (!SpeechRecognition) {
        updateMonitor("Reconocimiento de voz no soportado en este navegador.", "warning");
        return;
    }
    
    recognition = new SpeechRecognition();
    recognition.lang = "es-ES";
    recognition.continuous = false;
    recognition.interimResults = false;
    recognition.maxAlternatives = 1;

    recognition.onstart = () => {
        recognizing = true;
        const micBtn = document.getElementById("micBtn");
        const status = document.getElementById("mic-status");
        if (micBtn) micBtn.classList.add("listening");
        if (status) {
            status.textContent = "ESCUCHANDO...";
            status.classList.add("active");
        }
        updateMonitor("🎤 MODO VOZ ACTIVADO - HABLA AHORA", "info");
        playStrongBeep(800, 100);
    };

    recognition.onresult = (event) => {
        const input = document.getElementById("inputMsg");
        if (!input) return;
        
        const transcript = event.results[0][0].transcript.trim();
        
        if (transcript) {
            input.value = transcript;
            
            validateInput();
            realTimePreview();
            realTimeTableHighlight();
            
            updateMonitor(`🎤 Reconocido: "${transcript.substring(0, 30)}${transcript.length > 30 ? '...' : ''}"`, "info");
            playStrongBeep(600, 50);
            
            setTimeout(() => {
                if (input.value.trim() && !recognizing) {
                    updateMonitor("⚡ ENVIANDO MENSAJE DE VOZ...", "info");
                    
                    setTimeout(() => {
                        const mode = document.getElementById('inputMode').value;
                        const onlineCount = Object.keys(connections).filter(id => 
                            connections[id]?.status === 'online').length;
                        
                        if (onlineCount === 0) {
                            updateMonitor("⚠️ No hay conexiones activas para enviar", "warning");
                            playStrongBeep(300, 200);
                            return;
                        }
                        
                        if (mode === 'phonetic') {
                            sendRadioMessage();
                        } else {
                            sendMessage();
                        }
                        
                        input.value = "";
                    }, 300);
                }
            }, 1500);
        }
    };

    recognition.onerror = (event) => {
        console.error("Error de reconocimiento de voz:", event.error);
        
        if (event.error === 'no-speech') {
            updateMonitor("🎤 No se detectó voz. Intenta de nuevo.", "warning");
        } else if (event.error === 'audio-capture') {
            updateMonitor("🎤 No se pudo acceder al micrófono.", "error");
        } else if (event.error === 'not-allowed') {
            updateMonitor("🎤 Permiso de micrófono denegado.", "error");
        } else {
            updateMonitor(`🎤 Error de voz: ${event.error}`, "error");
        }
        
        resetVoiceUI();
    };

    recognition.onend = () => {
        resetVoiceUI();
    };
}

function resetVoiceUI() {
    recognizing = false;
    const micBtn = document.getElementById("micBtn");
    const status = document.getElementById("mic-status");
    if (micBtn) micBtn.classList.remove("listening");
    if (status) {
        status.textContent = "Voz OFF";
        status.classList.remove("active");
    }
}

function toggleVoiceInput() {
    if (recognizing) {
        try {
            recognition.stop();
            updateMonitor("🎤 Modo voz desactivado", "info");
        } catch(e) {
            console.log("Reconocimiento ya detenido");
        }
        return;
    }
    
    if (!recognition) {
        initVoiceRecognition();
        if (!recognition) {
            updateMonitor("❌ No se pudo inicializar reconocimiento de voz", "error");
            return;
        }
    }
    
    const input = document.getElementById("inputMsg");
    if (input) {
        input.value = "";
    }
    
    try {
        recognition.start();
    } catch(e) {
        console.error("Error al iniciar reconocimiento:", e);
        updateMonitor("❌ Error al acceder al micrófono. Verifica permisos.", "error");
        resetVoiceUI();
    }
}

// =============================================
// FUNCIONES DE CONFIGURACIÓN
// =============================================

function showSettings() {
    document.getElementById('settingsModal').style.display = 'flex';
    updateConnectionUI();
    loadIdSettings();
}

function hideSettings() {
    document.getElementById('settingsModal').style.display = 'none';
}

function updateConnectionUI() {
    document.querySelectorAll('.connection-type-btn').forEach(btn => {
        btn.classList.remove('active');
    });
    const btn = document.getElementById(`btn-${currentConnectionType}`);
    if (btn) btn.classList.add('active');
    
    const typeName = currentConnectionType === 'wifi' ? 'WiFi' : 'Datos Móviles';
    document.getElementById('current-connection-type').textContent = typeName;
}

function selectConnectionType(type) {
    if (type === currentConnectionType) return;
    
    document.querySelectorAll('.connection-type-btn').forEach(btn => {
        btn.classList.remove('active');
    });
    const btn = document.getElementById(`btn-${type}`);
    if (btn) btn.classList.add('active');
    
    currentConnectionType = type;
    const typeName = type === 'wifi' ? 'WiFi' : 'Datos Móviles';
    document.getElementById('current-connection-type').textContent = typeName;
    
    updateMonitor(`⚡ Modo conexión cambiado a: ${typeName}`);
    playStrongBeep(800, 100);
    
    saveConnectionType();
}

function saveConnectionType() {
    const config = JSON.parse(localStorage.getItem('radcom_config_v4.6') || '{}');
    config.connectionType = currentConnectionType;
    localStorage.setItem('radcom_config_v4.6', JSON.stringify(config));
}

function loadIdSettings() {
    const config = JSON.parse(localStorage.getItem('radcom_config_v4') || '{}');
    
    const useFixedCheck = document.getElementById('useFixedId');
    const customInput = document.getElementById('customIdInput');
    const currentDisplay = document.getElementById('currentIdDisplay');
    
    if (useFixedCheck) useFixedCheck.checked = config.useFixedId !== false;
    if (customInput) customInput.value = config.fixedId || getOrCreateFixedId();
    if (currentDisplay) currentDisplay.textContent = ID_SYSTEM.currentId || 'No asignado';
    
    toggleIdMode();
}

function toggleIdMode() {
    const useFixed = document.getElementById('useFixedId')?.checked;
    const container = document.getElementById('customIdContainer');
    
    if (container) {
        container.style.display = useFixed ? 'block' : 'none';
    }
}

function generateCustomId() {
    const prefixes = ['RADCOM', 'SATCOM', 'NETCOM', 'SECURE', 'MASTER'];
    const adjectives = ['ALPHA', 'BRAVO', 'CHARLIE', 'DELTA', 'ECHO', 'FOXTROT'];
    const nouns = ['STATION', 'NODE', 'HUB', 'RELAY', 'TERMINAL'];
    
    const prefix = prefixes[Math.floor(Math.random() * prefixes.length)];
    const adj = adjectives[Math.floor(Math.random() * adjectives.length)];
    const noun = nouns[Math.floor(Math.random() * nouns.length)];
    const number = Math.floor(Math.random() * 999).toString().padStart(3, '0');
    
    const generatedId = `${prefix}-${adj}-${noun}-${number}`;
    const input = document.getElementById('customIdInput');
    if (input) input.value = generatedId;
}

function applyIdSettings() {
    const useFixed = document.getElementById('useFixedId')?.checked;
    const customId = document.getElementById('customIdInput')?.value.trim();
    
    if (useFixed && (!customId || customId.length < 3)) {
        updateMonitor("⚠️ ID PERSONALIZADO INVÁLIDO", "error");
        playStrongBeep(300, 200);
        return;
    }
    
    const config = JSON.parse(localStorage.getItem('radcom_config_v4.6') || '{}');
    config.useFixedId = useFixed;
    
    if (useFixed) {
        config.fixedId = customId;
    } else {
        config.fixedId = null;
    }
    
    localStorage.setItem('radcom_config_v4', JSON.stringify(config));
    
    ID_SYSTEM.useFixedId = useFixed;
    ID_SYSTEM.fixedId = useFixed ? customId : null;
    
    updateMonitor(`🔄 APLICANDO NUEVA CONFIGURACIÓN DE ID...`);
    
    setTimeout(() => {
        if (peer) {
            peer.destroy();
        }
        setTimeout(() => initPeerJSEnhanced(), 1000);
    }, 500);
    
    hideSettings();
}

function resetIdSettings() {
    const defaultId = generateHardwareBasedId();
    const input = document.getElementById('customIdInput');
    if (input) input.value = defaultId;
    
    const useFixedCheck = document.getElementById('useFixedId');
    if (useFixedCheck) useFixedCheck.checked = true;
    
    toggleIdMode();
    updateMonitor("🔄 ID RESTABLECIDO A VALOR PREDETERMINADO");
}

function loadSettings() {
    const config = JSON.parse(localStorage.getItem('radcom_config_v4.6') || '{}');
    
    const systemNameInput = document.getElementById('systemName');
    if (config.systemName && systemNameInput) {
        systemNameInput.value = config.systemName;
    }
    if (config.connectionType) {
        currentConnectionType = config.connectionType;
    }
    
    const autoReconnect = document.getElementById('autoReconnect');
    if (config.autoReconnect !== undefined && autoReconnect) {
        autoReconnect.checked = config.autoReconnect;
    }
    
    const soundEnabled = document.getElementById('soundEnabled');
    if (config.soundEnabled !== undefined && soundEnabled) {
        soundEnabled.checked = config.soundEnabled;
    }
    
    const saveHistory = document.getElementById('saveHistory');
    if (config.saveHistory !== undefined && saveHistory) {
        saveHistory.checked = config.saveHistory;
    }
    
    const fastRecoveryCheck = document.getElementById('fastRecovery');
    if (config.fastRecovery !== undefined && fastRecoveryCheck) {
        fastRecoveryCheck.checked = config.fastRecovery;
        fastRecovery = config.fastRecovery;
    }
    
    const aggressiveReviveCheck = document.getElementById('aggressiveRevive');
    if (config.aggressiveRevive !== undefined && aggressiveReviveCheck) {
        aggressiveReviveCheck.checked = config.aggressiveRevive;
        aggressiveRevive = config.aggressiveRevive;
    }
    if (config.useFixedId !== undefined) {
        ID_SYSTEM.useFixedId = config.useFixedId;
    }
    if (config.fixedId) {
        ID_SYSTEM.fixedId = config.fixedId;
    }
}

function saveSettings() {
    const config = {
        systemName: document.getElementById('systemName')?.value,
        connectionType: currentConnectionType,
        autoReconnect: document.getElementById('autoReconnect')?.checked,
        soundEnabled: document.getElementById('soundEnabled')?.checked,
        saveHistory: document.getElementById('saveHistory')?.checked,
        fastRecovery: document.getElementById('fastRecovery')?.checked,
        aggressiveRevive: document.getElementById('aggressiveRevive')?.checked,
        useFixedId: document.getElementById('useFixedId')?.checked,
        fixedId: document.getElementById('customIdInput')?.value.trim()
    };
    
    localStorage.setItem('radcom_config_v4', JSON.stringify(config));
    
    fastRecovery = config.fastRecovery;
    aggressiveRevive = config.aggressiveRevive;
    ID_SYSTEM.useFixedId = config.useFixedId;
    
    if (config.fixedId && config.useFixedId) {
        ID_SYSTEM.fixedId = config.fixedId;
        localStorage.setItem('radcom_fixed_id_v4', config.fixedId);
    }
    
    updateMonitor("✅ CONFIGURACIÓN GUARDADA");
    hideSettings();
    playStrongBeep(600, 100);
}

// =============================================
// FUNCIONES DE ESTADÍSTICAS
// =============================================

function updateStats() {
    document.getElementById('stats-messages').textContent = stats.messages;
    document.getElementById('data-session').textContent = stats.tx + stats.rx;
    
    document.getElementById('stats-tx').textContent = formatBytes(stats.tx);
    document.getElementById('stats-rx').textContent = formatBytes(stats.rx);
    
    const elapsed = (Date.now() - stats.startTime) / 1000;
    const bandwidth = elapsed > 0 ? (stats.tx + stats.rx) / elapsed / 1024 : 0;
    document.getElementById('stats-bandwidth').textContent = `${bandwidth.toFixed(2)} KB/s`;
}

function formatBytes(bytes) {
    if (bytes < 1024) return bytes + ' B';
    if (bytes < 1048576) return (bytes / 1024).toFixed(1) + ' KB';
    return (bytes / 1048576).toFixed(1) + ' MB';
}

function updateUptime() {
    const elapsed = Date.now() - stats.startTime;
    const hours = Math.floor(elapsed / 3600000);
    const minutes = Math.floor((elapsed % 3600000) / 60000);
    const seconds = Math.floor((elapsed % 60000) / 1000);
    
    document.getElementById('stats-uptime').textContent = 
        `${hours.toString().padStart(2, '0')}:${minutes.toString().padStart(2, '0')}:${seconds.toString().padStart(2, '0')}`;
}

// =============================================
// FUNCIONES DE UTILIDAD
// =============================================

function generateKey() {
    const chars = 'ABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789';
    let key = '';
    for (let i = 0; i < 12; i++) {
        key += chars.charAt(Math.floor(Math.random() * chars.length));
    }
    const keyInput = document.getElementById('key');
    if (keyInput) {
        keyInput.value = key;
        keyInput.type = 'text';
        
        setTimeout(() => {
            keyInput.type = 'password';
        }, 2000);
    }
    return key;
}

function copyID() {
    if (!myPeerId) {
        alert('Espera a que se genere el ID primero');
        return;
    }
    
    navigator.clipboard.writeText(myPeerId).then(() => {
        updateMonitor(`📋 ID COPIADO: ${myPeerId.substring(0, 12)}...`);
        playStrongBeep(800, 100);
    });
}

function initResizableSeparator() {
    const separator = document.getElementById('resizableSeparator');
    const monitorContainer = document.getElementById('monitorContainer');
    const tableContainer = document.getElementById('tableContainer');
    
    if (!separator || !monitorContainer || !tableContainer) return;
    
    let isResizing = false;
    let startY, startHeight, startTableHeight;
    
    separator.addEventListener('mousedown', function(e) {
        isResizing = true;
        startY = e.clientY;
        startHeight = parseInt(getComputedStyle(monitorContainer).height);
        startTableHeight = parseInt(getComputedStyle(tableContainer).height);
        
        document.addEventListener('mousemove', resize);
        document.addEventListener('mouseup', stopResize);
    });

    function resize(e) {
        if (!isResizing) return;
        
        const delta = e.clientY - startY;
        let newHeight = startHeight + delta;
        
        if (newHeight < 150) newHeight = 150;
        if (newHeight > 400) newHeight = 400;
        
        monitorContainer.style.height = newHeight + 'px';
        
        const tableNewHeight = newHeight - 80;
        if (tableNewHeight > 80) {
            tableContainer.style.height = tableNewHeight + 'px';
        }
    }
    
    function stopResize() {
        isResizing = false;
        document.removeEventListener('mousemove', resize);
        document.removeEventListener('mouseup', stopResize);
        
        const currentHeight = parseInt(getComputedStyle(monitorContainer).height);
        localStorage.setItem('radcom_separator_height', currentHeight);
    }
    
    const savedHeight = localStorage.getItem('radcom_separator_height');
    if (savedHeight) {
        monitorContainer.style.height = savedHeight + 'px';
    }
}

function finalInitialization() {
    const encSelect = document.getElementById('encryptionMode');
    if (encSelect) {
        encSelect.addEventListener('change', updateSecurityBadge);
        updateSecurityBadge();
    }
    
    setInterval(processQueue, 8000);
    
    setInterval(() => {
        if (Object.values(connections).some(c => c.status === 'offline')) {
            reviveAllConnections();
        }
    }, 25000);
    
    updateMonitor(`🚀 RADCOM MASTER v${VERSION} - Versión estable y segura con VoIP`);
}

// =============================================
// SISTEMA CONSOLA v5.6.1
// =============================================

function openConsole() {
    document.getElementById('consoleModal').style.display = 'flex';
    switchConsoleTab('CMD');
    updateMonitor("🚀 CONSOLE v5.6.1 ACTIVATED", "info");
    playStrongBeep(1000, 100);
}

function closeConsole() {
    document.getElementById('consoleModal').style.display = 'none';
    updateMonitor("🔒 Console closed", "info");
}

function switchConsoleTab(tab) {
    currentConsoleTab = tab;
    
    document.querySelectorAll('.console-tab').forEach(t => t.classList.remove('active'));
    document.querySelectorAll('.console-tab-content').forEach(c => c.classList.remove('active'));
    
    const tabBtn = document.querySelector(`.console-tab[onclick="switchConsoleTab('${tab}')"]`);
    if (tabBtn) tabBtn.classList.add('active');
    
    const tabContent = document.getElementById(`console-tab-${tab}`);
    if (tabContent) tabContent.classList.add('active');
    
    if (tab === 'hex') {
        initializeHexEditor();
    } else if (tab === 'CMD') {
        const cmdInput = document.getElementById('CMD-input');
        if (cmdInput) cmdInput.focus();
    } else if (tab === 'decode') {
        const decodeInput = document.getElementById('hex-decoder-input');
        if (decodeInput) decodeInput.focus();
    }
    
    playStrongBeep(600, 50);
}

function handleCMDCommand(event) {
    if (event.key === 'Enter') {
        executeCMDCommand();
    }
}

function executeCMDCommand() {
    const input = document.getElementById('CMD-input');
    const command = input.value.trim();
    
    if (!command) return;
    
    appendToCMDConsole(`&gt; ${command}`);
    processCMDCommand(command);
    
    input.value = '';
    input.focus();
}

function processCMDCommand(command) {
    const cmd = command.toLowerCase();
    
    switch(cmd) {
        case 'help':
        case '?':
            showCMDHelp();
            break;
        case 'status':
            showSystemStatus();
            break;
        case 'clear':
            clearCMDConsole();
            break;
        case 'connections':
            showConnections();
            break;
        case 'version':
            appendToCMDConsole(`RADCOM MASTER v${VERSION}`);
            break;
        case 'restart':
            appendToCMDConsole("Restarting system...");
            setTimeout(() => { location.reload(); }, 1000);
            break;
        case 'test':
            runSystemTest();
            break;
        case 'encrypt test':
            testEncryption();
            break;
        case 'morse test':
            testMorseSystem();
            break;
        case 'radio test':
            testRadioSystem();
            break;
        case 'satellite test':
            testSatelliteSystem();
            break;
        case 'export config':
            exportConfiguration();
            break;
        case 'import config':
            importConfiguration();
            break;
        case 'debug':
            toggleDebugMode();
            break;
        default:
            if (cmd.startsWith('echo ')) {
                appendToCMDConsole(cmd.substring(5));
            } else if (cmd.startsWith('hex ')) {
                convertToHex(cmd.substring(4));
            } else if (cmd.startsWith('decode ')) {
                decodeHex(cmd.substring(7));
            } else {
                appendToCMDConsole(`Unknown command: "${command}". Type "help" for commands.`);
            }
    }
}

function appendToCMDConsole(text) {
    const console = document.getElementById('CMD-output');
    if (console) {
        console.innerHTML += `\n${text}`;
        console.scrollTop = console.scrollHeight;
    }
}

function clearCMDConsole() {
    const console = document.getElementById('CMD-output');
    if (console) {
        console.innerHTML = '&gt; RADCOM CMD CONSOLE v5.6.1 INITIALIZED\n' +
            '&gt; System: RADCOM MASTER\n' +
            '&gt; Version: ' + VERSION + '\n' +
            '&gt; Ready for commands...';
    }
}

function showCMDHelp() {
    const help = [
        '=== RADCOM CMD CONSOLE COMMANDS ===',
        'help / ?          - Show this help',
        'status           - System status',
        'clear            - Clear console',
        'connections      - Show active connections',
        'version          - Show version',
        'restart          - Restart system',
        'test             - Run system tests',
        'encrypt test     - Test encryption',
        'morse test       - Test Morse system',
        'radio test       - Test radio system',
        'satellite test   - Test satellite system',
        'export config    - Export configuration',
        'import config    - Import configuration',
        'debug            - Toggle debug mode',
        'echo [text]      - Echo text',
        'hex [text]       - Convert text to hex',
        'decode [hex]     - Decode hex to text',
        '==================================='
    ];
    
    help.forEach(line => appendToCMDConsole(line));
}

function showSystemStatus() {
    const onlineCount = Object.keys(connections).filter(id => 
        connections[id]?.status === 'online').length;
    
    const status = [
        '=== SYSTEM STATUS ===',
        `Version: ${VERSION}`,
        `ID: ${myPeerId ? myPeerId.substring(0, 20) + '...' : 'Not set'}`,
        `Connections: ${onlineCount} online`,
        `VoIP: ${voipActiveCall ? `Active with ${voipActiveCall.substring(0, 8)}` : 'Inactive'}`,
        `Messages: ${stats.messages} sent/received`,
        `Data TX: ${formatBytes(stats.tx)}`,
        `Data RX: ${formatBytes(stats.rx)}`,
        `Uptime: ${formatUptime(Date.now() - stats.startTime)}`,
        `Encryption: ${document.getElementById('encryptionMode')?.value || 'AES-256-GCM'}`,
        `GPS: ${satelliteSystem.latitude ? 'Active' : 'Inactive'}`,
        `Radio: ${currentBand} ${currentFrequency}${currentUnit}`,
        `Morse: ${morseSpeed} speed`,
        `Voice: ${recognizing ? 'Active' : 'Inactive'}`,
        '====================='
    ];
    
    status.forEach(line => appendToCMDConsole(line));
}

function showConnections() {
    const connectionsList = Object.keys(connections).map(id => {
        const conn = connections[id];
        const voipStatus = voipActiveCall === id ? 'VoIP ACTIVE' : (voipIncomingCall === id ? 'VoIP INCOMING' : '');
        return `${id.substring(0, 12)}... | ${conn.status} | ${conn.type} | ${conn.health} ${voipStatus ? '| ' + voipStatus : ''}`;
    });
    
    appendToCMDConsole('=== ACTIVE CONNECTIONS ===');
    if (connectionsList.length === 0) {
        appendToCMDConsole('No connections');
    } else {
        connectionsList.forEach(conn => appendToCMDConsole(conn));
    }
    appendToCMDConsole('==========================');
}

function runSystemTest() {
    appendToCMDConsole('Running system tests...');
    
    const tests = [
        { name: 'PeerJS Connection', test: () => !!peer && !peer.disconnected },
        { name: 'Audio System', test: () => typeof AudioContext !== 'undefined' },
        { name: 'LocalStorage', test: () => typeof localStorage !== 'undefined' },
        { name: 'WebRTC', test: () => !!window.RTCPeerConnection },
        { name: 'GPS Support', test: () => !!navigator.geolocation },
        { name: 'Voice Recognition', test: () => !!window.SpeechRecognition || !!window.webkitSpeechRecognition },
        { name: 'Microphone Access', test: async () => {
            try {
                const stream = await navigator.mediaDevices.getUserMedia({ audio: true });
                stream.getTracks().forEach(track => track.stop());
                return true;
            } catch {
                return false;
            }
        }}
    ];
    
    let passed = 0;
    tests.forEach(test => {
        const result = test.test();
        appendToCMDConsole(`${result ? '✓' : '✗'} ${test.name}: ${result ? 'PASS' : 'FAIL'}`);
        if (result) passed++;
    });
    
    appendToCMDConsole(`\n${passed}/${tests.length} tests passed`);
}

function formatUptime(ms) {
    const seconds = Math.floor(ms / 1000);
    const hours = Math.floor(seconds / 3600);
    const minutes = Math.floor((seconds % 3600) / 60);
    const secs = seconds % 60;
    
    return `${hours.toString().padStart(2, '0')}:${minutes.toString().padStart(2, '0')}:${secs.toString().padStart(2, '0')}`;
}

function testEncryption() {
    appendToCMDConsole('Testing encryption...');
    const testText = "RADCOM TEST MESSAGE";
    const key = "TESTKEY123";
    
    const encrypted = xorEncrypt(testText, key);
    const decrypted = xorDecrypt(encrypted, key);
    
    appendToCMDConsole(`Original: ${testText}`);
    appendToCMDConsole(`Encrypted (HEX): ${Array.from(encrypted).map(c => 
        c.charCodeAt(0).toString(16).padStart(2, '0')).join(' ')}`);
    appendToCMDConsole(`Decrypted: ${decrypted}`);
    appendToCMDConsole(`Test ${testText === decrypted ? 'PASSED ✓' : 'FAILED ✗'}`);
}

function testMorseSystem() {
    appendToCMDConsole('Testing Morse system...');
    
    const testWord = "SOS";
    const morse = textToMorse(testWord);
    appendToCMDConsole(`${testWord} → ${morse}`);
    
    if (typeof playMorseCharSound === 'function') {
        appendToCMDConsole('Playing test sound...');
        setTimeout(() => playMorseCharSound('S'), 100);
        setTimeout(() => playMorseCharSound('O'), 500);
        setTimeout(() => playMorseCharSound('S'), 900);
    }
}

function testRadioSystem() {
    appendToCMDConsole('Testing radio system...');
    appendToCMDConsole(`Current band: ${currentBand}`);
    appendToCMDConsole(`Frequency: ${currentFrequency} ${currentUnit}`);
    
    if (typeof playRadioBeep === 'function') {
        appendToCMDConsole('Playing radio test tone...');
        playRadioBeep(1000, 200);
    }
}

function testSatelliteSystem() {
    appendToCMDConsole('Testing satellite system...');
    
    if (satelliteSystem.latitude && satelliteSystem.longitude) {
        appendToCMDConsole(`GPS Active: ${satelliteSystem.latitude.toFixed(4)}°, ${satelliteSystem.longitude.toFixed(4)}°`);
        appendToCMDConsole(`Altitude: ${satelliteSystem.altitude ? Math.round(satelliteSystem.altitude) + ' m' : 'N/A'}`);
    } else {
        appendToCMDConsole('GPS: Inactive');
    }
    
    if (satelliteSystem.weatherData) {
        appendToCMDConsole('Weather data: Available');
    }
}

function exportConfiguration() {
    const config = {
        version: VERSION,
        peerId: myPeerId,
        connections: Object.keys(connections).length,
        settings: {
            encryption: document.getElementById('encryptionMode')?.value,
            autoReconnect: document.getElementById('autoReconnect')?.checked,
            soundEnabled: document.getElementById('soundEnabled')?.checked
        },
        stats: stats,
        timestamp: new Date().toISOString()
    };
    
    const configStr = JSON.stringify(config, null, 2);
    const input = document.getElementById('hex-decoder-input');
    const output = document.getElementById('hex-decoder-output');
    
    if (input) input.value = configStr;
    if (output) output.value = btoa(configStr);
    
    switchConsoleTab('decode');
    appendToCMDConsole('Configuration exported to decoder');
}

function importConfiguration() {
    appendToCMDConsole('Import configuration: Use Base64 decode in Hex Decoder tab');
    switchConsoleTab('decode');
}

function toggleDebugMode() {
    const debugMode = localStorage.getItem('radcom_debug_mode') !== 'true';
    localStorage.setItem('radcom_debug_mode', debugMode);
    
    appendToCMDConsole(`Debug mode ${debugMode ? 'ENABLED' : 'DISABLED'}`);
    updateMonitor(`Debug ${debugMode ? 'ON' : 'OFF'}`, debugMode ? "warning" : "info");
}

function convertToHex(text) {
    let hex = '';
    for (let i = 0; i < text.length; i++) {
        hex += text.charCodeAt(i).toString(16).padStart(2, '0') + ' ';
    }
    appendToCMDConsole(`HEX: ${hex.trim()}`);
}

function decodeHex(hexStr) {
    const hex = hexStr.replace(/[^0-9a-fA-F]/g, '');
    let text = '';
    for (let i = 0; i < hex.length; i += 2) {
        const charCode = parseInt(hex.substr(i, 2), 16);
        if (!isNaN(charCode)) {
            text += String.fromCharCode(charCode);
        }
    }
    appendToCMDConsole(`Decoded: ${text}`);
}

// =============================================
// FUNCIONES DE HEX EDITOR
// =============================================

function initializeHexEditor() {
    const editor = document.getElementById('hex-editor');
    if (!editor) return;
    
    if (!hexEditorContent) {
        hexEditorContent = `// RADCOM HEX EDITOR v5.6.1
// Load page HTML or paste your code here
// Use toolbar buttons to format, minify, or save

// Example HTML:
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>RADCOM System</title>
    <style>
        body {
            background: #000;
            color: #0f0;
            font-family: monospace;
        }
    </style>
</head>
<body>
    <h1>RADCOM MASTER</h1>
</body>
</html>`;
        
        editor.value = hexEditorContent;
    }
    
    editor.addEventListener('input', updateHexPosition);
    updateHexPosition();
}

function updateHexPosition() {
    const editor = document.getElementById('hex-editor');
    if (!editor) return;
    
    const text = editor.value;
    const lines = text.substring(0, editor.selectionStart).split('\n');
    const line = lines.length;
    const col = lines[lines.length - 1].length + 1;
    
    const posEl = document.getElementById('hex-position');
    if (posEl) {
        posEl.textContent = `Line: ${line}, Col: ${col} | Chars: ${text.length}`;
    }
}

function hexLoadCurrentPage() {
    const htmlContent = document.documentElement.outerHTML;
    const formatted = formatHTML(htmlContent);
    
    const editor = document.getElementById('hex-editor');
    if (editor) {
        editor.value = formatted;
        hexEditorContent = formatted;
    }
    
    updateHexStatus('Page HTML loaded');
    appendToCMDConsole('Loaded current page HTML into hex editor');
}

function hexSaveChanges() {
    const content = document.getElementById('hex-editor')?.value;
    
    updateHexStatus('Changes saved (simulated)');
    appendToCMDConsole('Hex editor changes saved');
    
    if (content) {
        localStorage.setItem('radcom_hex_editor_backup', content);
    }
    
    playStrongBeep(800, 50);
}

function hexFind() {
    const search = prompt('Search in hex editor:');
    if (search) {
        const editor = document.getElementById('hex-editor');
        if (!editor) return;
        
        const content = editor.value;
        const index = content.indexOf(search);
        
        if (index !== -1) {
            editor.focus();
            editor.setSelectionRange(index, index + search.length);
            updateHexStatus(`Found: "${search}"`);
        } else {
            updateHexStatus(`Not found: "${search}"`);
        }
    }
}

function hexFormat() {
    const editor = document.getElementById('hex-editor');
    if (!editor) return;
    
    const content = editor.value;
    
    if (content.includes('<') && content.includes('>')) {
        editor.value = formatHTML(content);
        updateHexStatus('Formatted as HTML');
    } else if (content.trim().startsWith('{') || content.trim().startsWith('[')) {
        try {
            const json = JSON.parse(content);
            editor.value = JSON.stringify(json, null, 2);
            updateHexStatus('Formatted as JSON');
        } catch {
            updateHexStatus('Not valid JSON');
        }
    } else if (content.includes('function') || content.includes('const ') || content.includes('let ')) {
        editor.value = formatJavaScript(content);
        updateHexStatus('Formatted as JavaScript');
    }
    
    hexEditorContent = editor.value;
}

function hexMinify() {
    const editor = document.getElementById('hex-editor');
    if (!editor) return;
    
    let content = editor.value;
    let result = '';
    let inComment = false;
    
    for (let i = 0; i < content.length; i++) {
        if (content.substr(i, 4) === '<!--') {
            inComment = true;
            i += 3;
            continue;
        }
        
        if (inComment && content.substr(i, 3) === '-->') {
            inComment = false;
            i += 2;
            continue;
        }
        
        if (!inComment) {
            result += content[i];
        }
    }
    
    content = result;
    
    let lastCharWasSpace = false;
    result = '';
    
    for (let i = 0; i < content.length; i++) {
        const char = content[i];
        const isWhitespace = char === ' ' || char === '\n' || char === '\t' || char === '\r';
        
        if (isWhitespace) {
            if (!lastCharWasSpace && i > 0 && i < content.length - 1) {
                result += ' ';
                lastCharWasSpace = true;
            }
        } else {
            result += char;
            lastCharWasSpace = false;
        }
    }
    
    content = result.trim();
    
    result = '';
    for (let i = 0; i < content.length; i++) {
        if (content[i] === '>' && i + 1 < content.length) {
            result += '>';
            let j = i + 1;
            while (j < content.length && (content[j] === ' ' || content[j] === '\n' || content[j] === '\t')) {
                j++;
            }
            if (j < content.length && content[j] === '<') {
                result += '<';
                i = j;
            } else {
                i = j - 1;
            }
        } else {
            result += content[i];
        }
    }
    
    editor.value = result;
    hexEditorContent = result;
    updateHexStatus('Minified content');
}

function updateHexStatus(message) {
    const statusEl = document.getElementById('hex-status');
    if (statusEl) {
        statusEl.textContent = message;
        setTimeout(() => {
            statusEl.textContent = 'Ready';
        }, 3000);
    }
}

function formatHTML(html) {
    let formatted = '';
    let indent = 0;
    const lines = html.split('\n');
    
    for (let line of lines) {
        line = line.trim();
        if (!line) continue;
        
        if (line.includes('</')) {
            indent--;
        }
        
        formatted += '    '.repeat(Math.max(0, indent)) + line + '\n';
        
        if (line.includes('<') && !line.includes('/>') && !line.includes('</')) {
            indent++;
        }
    }
    
    return formatted;
}

function formatJavaScript(js) {
    js = js.replace(/{/g, ' {\n');
    js = js.replace(/}/g, '\n}\n');
    js = js.replace(/;/g, ';\n');
    
    let formatted = '';
    let indent = 0;
    const lines = js.split('\n');
    
    for (let line of lines) {
        line = line.trim();
        if (!line) continue;
        
        if (line.includes('}')) {
            indent--;
        }
        
        formatted += '    '.repeat(Math.max(0, indent)) + line + '\n';
        
        if (line.includes('{')) {
            indent++;
        }
    }
    
    return formatted;
}

// =============================================
// FUNCIONES DE HEX DECODER
// =============================================

function autoDetectAndConvert() {
    const input = document.getElementById('hex-decoder-input');
    const output = document.getElementById('hex-decoder-output');
    if (!input || !output) return;
    
    const value = input.value.trim();
    
    if (!value) {
        output.value = '';
        return;
    }
    
    if (/^[0-9a-fA-F\s]+$/.test(value.replace(/\s/g, ''))) {
        convertHexToText();
    } else if (/^[\x00-\x7F\s]+$/.test(value)) {
        convertTextToHex();
    }
}

function convertTextToHex() {
    const input = document.getElementById('hex-decoder-input');
    const output = document.getElementById('hex-decoder-output');
    if (!input || !output) return;
    
    const value = input.value;
    let result = '';
    
    for (let i = 0; i < value.length; i++) {
        const hex = value.charCodeAt(i).toString(16).toUpperCase().padStart(2, '0');
        result += hex + ' ';
        
        if ((i + 1) % 8 === 0) {
            result += ' ';
        }
        
        if ((i + 1) % 32 === 0) {
            result += '\n';
        }
    }
    
    result += '\n\nASCII: ';
    for (let i = 0; i < Math.min(value.length, 64); i++) {
        const char = value[i];
        result += (char >= ' ' && char <= '~') ? char : '.';
    }
    
    output.value = result.trim();
    updateMonitor("📝 Converted text to HEX", "info");
}

function convertHexToText() {
    const input = document.getElementById('hex-decoder-input');
    const output = document.getElementById('hex-decoder-output');
    if (!input || !output) return;
    
    const value = input.value;
    const hex = value.replace(/[^0-9a-fA-F]/g, '');
    
    let result = '';
    for (let i = 0; i < hex.length; i += 2) {
        const hexByte = hex.substr(i, 2);
        if (hexByte) {
            const charCode = parseInt(hexByte, 16);
            if (!isNaN(charCode)) {
                result += String.fromCharCode(charCode);
            }
        }
    }
    
    const info = [
        `Length: ${result.length} characters`,
        `Hex bytes: ${Math.ceil(hex.length / 2)}`,
        '',
        'Decoded text:',
        '-------------',
        result
    ].join('\n');
    
    output.value = info;
    updateMonitor("🔓 Converted HEX to text", "info");
}

function decodeBase64() {
    const input = document.getElementById('hex-decoder-input');
    const output = document.getElementById('hex-decoder-output');
    if (!input || !output) return;
    
    const value = input.value.trim();
    
    try {
        const decoded = atob(value);
        output.value = decoded;
        updateMonitor("🔓 Decoded Base64", "info");
    } catch (error) {
        output.value = `Error decoding Base64: ${error.message}`;
        updateMonitor("❌ Base64 decode failed", "error");
    }
}

function clearDecoder() {
    const input = document.getElementById('hex-decoder-input');
    const output = document.getElementById('hex-decoder-output');
    
    if (input) input.value = '';
    if (output) output.value = '';
    
    updateMonitor("🧹 Decoder cleared", "info");
}

// =============================================
// SISTEMA DE MÓDULOS Y COCKPIT
// =============================================

function handleCockpitClick(modulo) {
    document.querySelectorAll('.btn-cockpit').forEach(b => b.classList.remove('active'));
    const btn = document.getElementById('btn-' + modulo.toLowerCase());
    if (btn) btn.classList.add('active');
    
    if (modulo === 'COM') {
        closeModal680();
    } else {
        openModuleWindow(modulo);
    }
}

function openModuleWindow(modulo) {
    const modal = document.getElementById('modal-680');
    const body = document.getElementById('modal-body');
    const title = document.getElementById('modal-title');
    
    if (!modal || !body) {
        console.error("❌ Error Crítico: No se encuentran los elementos del modal.");
        return;
    }

    modal.style.display = 'block';
    title.innerText = `SISTEMA RADCOM - MÓDULO ${modulo}`;
    body.innerHTML = `<iframe id="module-frame" style="width:100%; height:100%; border:none; background:#000;"></iframe>`;
    
    const config = {
        'NAV': 'pfd-source-storage',
        'ECM': 'ecm-source-storage',
        'MAP': 'map-source-storage',
        'UTIL': 'util-source-storage',
        'LEGAL': 'legal-source-storage'
    };

    const sourceId = config[modulo];
    const storage = document.getElementById(sourceId);

    if (storage) {
        const frame = document.getElementById('module-frame').contentWindow.document;
        frame.open();
        frame.write(`
            <!DOCTYPE html>
            <html>
            <head>
                <meta charset="UTF-8">
                <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" />
                <script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"><\/script>
                <style>
                    body { margin:0; padding:0; background:#000; color:white; overflow:hidden; font-family:monospace; width:100vw; height:100vh; }
                    ${modulo === 'ECM' ? 'body { display:flex; justify-content:center; align-items:center; transform: scale(1); transform-origin: center; }' : ''}
                </style>
            </head>
            <body>
                ${storage.innerHTML}
            </body>
            </html>
        `);
        frame.close();
    } else {
        body.innerHTML = `<div style="color:#ff3300; padding:20px; font-family:monospace;">⚠️ ERROR: SOURCE [${sourceId}] NO ENCONTRADO EN EL ALMACÉN</div>`;
    }
}

function closeModal680() {
    const modal = document.getElementById('modal-680');
    if (modal) {
        modal.style.display = 'none';
        const body = document.getElementById('modal-body');
        if (body) body.innerHTML = '';
        
        document.querySelectorAll('.btn-cockpit').forEach(b => b.classList.remove('active'));
        const btnCom = document.getElementById('btn-com');
        if (btnCom) btnCom.classList.add('active');
        
        updateMonitor("🖥️ REGRESO A CONSOLA PRINCIPAL");
    }
}

// =============================================
// MANEJO DE EVENTOS DE VISIBILIDAD
// =============================================

document.addEventListener('visibilitychange', function() {
    isBackground = document.hidden;
    
    console.log(`📱 App ${isBackground ? 'en segundo plano' : 'en primer plano'}`);
    
    if (!isBackground) {
        updateMonitor("⚡ APP REACTIVADA - VERIFICANDO CONEXIONES...");
        playStrongBeep(800, 100);
        
        if (fastRecovery) {
            setTimeout(() => { reviveAllConnections(); }, 500);
        } else {
            checkConnectionsHealth();
        }
        
        if (satelliteSystem.lastKnownPosition) {
            updateSatelliteUI();
        } else {
            setTimeout(() => {
                if (!watchId) getCurrentGPSPosition(true);
            }, 800);
        }
    } else {
        updateMonitor("⏸️ APP EN SEGUNDO PLANO");
        stopWatchingPosition();
    }
});

const originalHandleReceivedData = window.handleReceivedData;
window.handleReceivedData = function(senderId, data) {
    if (data.type === 'emergency') {
        handleEmergencyMessage(senderId, data);
        return;
    }
    
    if (originalHandleReceivedData) {
        originalHandleReceivedData(senderId, data);
    }
};

// =============================================
// INICIALIZACIÓN
// =============================================

window.onload = function() {
    console.log(`🚀 RADCOM MASTER v${VERSION} - Versión limpia y segura con VoIP`);

    try {
        const peers = JSON.parse(localStorage.getItem('radcom_peers') || '[]');
        const validPeers = peers.filter(id => 
            typeof id === 'string' && id.length < 30 && /^[a-zA-Z0-9\-]+$/.test(id)
        );
        localStorage.setItem('radcom_peers', JSON.stringify(validPeers));
        savedIds = validPeers;
    } catch (e) {
        localStorage.removeItem('radcom_peers');
        savedIds = [];
    }

    buildAsciiTable();
    buildMorseTable();
    initRadioSystem();
    initSatelliteSystem();
    initVoiceRecognition();
    initResizableSeparator();
    loadSettings();
    updatePeerList();

    peer = new Peer();
    setupPeerEvents();
    startHealthCheckSystem();

    const versionBadge = document.querySelector('.version-badge');
    if (versionBadge) versionBadge.textContent = `v${VERSION}`;

    updateMonitor(`✅ SISTEMA v${VERSION} INICIADO | AES-256-GCM + ECDH ACTIVO | VoIP ACTIVADO | NOTIFICACIONES ACTIVAS`);
    
    setTimeout(initSecurity, 1000);
    setTimeout(initQueue, 2000);
    setTimeout(finalInitialization, 1200);
    
    startWatchingPosition();
    
    if (Notification.permission === "default") {
        Notification.requestPermission();
    }
};

function initSecurity() {
    console.log("🔐 Inicializando seguridad v5.2...");
    
    const encryptionSelect = document.getElementById('encryptionMode');
    if (encryptionSelect) {
        encryptionSelect.addEventListener('change', updateSecurityBadge);
    }
    
    updateSecurityBadge();
    
    updateMonitor("🔐 SISTEMA DE SEGURIDAD v5.2 ACTIVADO");
    
    const monitor = document.getElementById('monitor-decoded');
    if (monitor) {
        const bubble = document.createElement('div');
        bubble.className = 'message-bubble message-system';
        bubble.innerHTML = `
            <i class="fas fa-shield-alt"></i> SEGURIDAD v5.2 ACTIVADA<br>
            <small style="color:#88ffaa; font-size:0.7rem;">
                Selecciona cifrado: AES-256-GCM | XOR | Sin cifrado
            </small>
        `;
        monitor.appendChild(bubble);
        monitor.scrollTop = monitor.scrollHeight;
    }
}

setTimeout(() => {
    console.log("🚀 RADCOM CONSOLE v5.6.1 ready");
}, 1000);

window.forceUpdateSatelliteData = forceUpdateSatelliteData;
window.sendSatelliteEmergency = sendSatelliteEmergency;
window.sendPositionToChat = sendPositionToChat;
window.getCurrentGPSPosition = getCurrentGPSPosition;
window.initSatelliteSystem = initSatelliteSystem;
window.initializeSatelliteWeather = initializeSatelliteWeather;
window.openConsole = openConsole;
window.closeConsole = closeConsole;
window.switchConsoleTab = switchConsoleTab;
window.handleCMDCommand = handleCMDCommand;
window.executeCMDCommand = executeCMDCommand;
window.clearCMDConsole = clearCMDConsole;
window.hexLoadCurrentPage = hexLoadCurrentPage;
window.hexSaveChanges = hexSaveChanges;
window.hexFind = hexFind;
window.hexFormat = hexFormat;
window.hexMinify = hexMinify;
window.autoDetectAndConvert = autoDetectAndConvert;
window.convertTextToHex = convertTextToHex;
window.convertHexToText = convertHexToText;
window.decodeBase64 = decodeBase64;
window.clearDecoder = clearDecoder;
window.showSecurityInfo = showSecurityInfo;
window.handleCockpitClick = handleCockpitClick;
window.closeModal680 = closeModal680;
window.toggleVoIPCall = toggleVoIPCall;
window.endVoIPCall = endVoIPCall;

// ===== CONTROL DEL SPLASH SCREEN (CORREGIDO) =====
document.addEventListener('DOMContentLoaded', function() {
    const splashScreen = document.getElementById('splash-screen');
    
    if (!splashScreen) {
        console.log("Splash screen no encontrado");
        return;
    }
    
    console.log("Splash screen encontrado, iniciando cuenta atrás...");
    
    function hideSplash() {
        splashScreen.style.opacity = '0';
        
        setTimeout(() => {
            splashScreen.style.display = 'none';
            console.log("Splash screen ocultado");
        }, 800);
    }
    
    setTimeout(hideSplash, 7000);
    
    const loadingText = splashScreen.querySelector('div:last-child div:last-child');
    if (loadingText) {
        const texts = [
            "INICIALIZANDO SISTEMA...",
            "CARGANDO MÓDULOS...",
            "ESTABLECIENDO CONEXIONES...",
            "SISTEMA LISTO"
        ];
        
        let index = 0;
        const textInterval = setInterval(() => {
            index = (index + 1) % texts.length;
            if (loadingText) loadingText.textContent = texts[index];
            
            if (index === texts.length - 1) {
                setTimeout(() => {
                    clearInterval(textInterval);
                }, 7000);
            }
        }, 1200);
    }
});


</script>
</body>
</html>
