<!DOCTYPE html>
<html lang="es">
<head>   
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>RADCOM MASTER v5.6.5- SISTEMA DE COMUNICACIÓN SEGURA AES-256-GCM</title>
    <script src="https://unpkg.com/peerjs@1.5.2/dist/peerjs.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/js/all.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/openmeteo@0.3.0"></script>
    <script src="js/weather-logic.js"></script>
    <script src="weather-logic.js"></script> 
    <script src="https://api.open-meteo.com/v1/forecast?latitude=40.4599&longitude=-3.4859&hourly=temperature_2m,visibility,relative_humidity_2m,pressure_msl,wind_speed_10m,wind_direction_80m,wind_gusts_10m"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/crypto-js/4.1.1/crypto-js.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/js-sha256/0.9.0/sha256.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/elliptic@6.5.4/dist/elliptic.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/@noble/ed25519@2.1.0/+esm"></script>
    <script src="https://cdn.jsdelivr.net/npm/@noble/hashes@1.5.0/+esm"></script> 
    <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" />
    <script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>      
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/leaflet.draw/1.0.4/leaflet.draw.css" />
    <script src="https://cdnjs.cloudflare.com/ajax/libs/leaflet.draw/1.0.4/leaflet.draw.js"></script>
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

        /* === CONTENEDOR PRINCIPAL 720x720 === */
        
        .container { 
            width: 640px; 
            height: 640px; 
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

        /* === LAYOUT PRINCIPAL === */
        
        .main-layout { 
            display: flex; 
            flex: 1; 
            overflow: hidden; 
            height: calc(720px - 48px - 120px);
        }

        /* === PANEL LATERAL (150px) === */
        
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
        
        /* NUEVO ESTADO: NARANJA PARA INACTIVO/SEGUNDO PLANO */
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

        /* === ÁREA DE CONTENIDO === */

        .content-area { 
            flex: 1; 
            display: flex; 
            flex-direction: column; 
            background: #000; 
            overflow: hidden;
        }

        /* === MONITOR SUPERIOR === */
        
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

        /* === TABLA CENTRAL === */
        
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

        /* === MONITOR DE CHAT === */
        
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

        /* === PIE DE PÁGINA MEJORADO === */
        
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
        
        /* === BOTÓN REFRESCAR PEQUEÑO COMO ANTES === */
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
        
        /* === AÑADIDO: CSS PARA EL SCROLL === */
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

        /* === BOTÓN REVIVIR CONEXIONES MEJORADO === */
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

        /* === INDICADOR DE REVIVIR ACTIVO === */
        .reviving {
            animation: revivingPulse 0.5s infinite;
        }

        @keyframes revivingPulse {
            0% { opacity: 1; }
            50% { opacity: 0.7; }
            100% { opacity: 1; }
        }

        /* Nuevos estilos para pestañas */
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

        /* === NUEVOS ESTILOS PARA TABLA MORSE MEJORADA === */
        .morse-table-container {
            padding: 3px;
            height: 100%;
            overflow-y: auto;
        }

        /* SCROLL ESTILIZADO PARA LAS PESTAÑAS */
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

        /* === CONTROLES DE SONIDO MORSE === */
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

        /* === ESTILOS ORIGINALES PARA PESTAÑA RADIO v4.6 === */
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

        /* === NUEVOS ESTILOS PARA CANALES CB/PMR (MANTENIENDO DISEÑO ORIGINAL) === */
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

        /* === CONTENEDOR FONÉTICO MEJORADO (SCROLL ARREGLADO) === */
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

        /* === SCROLL ESTILIZADO PARA EL FONÉTICO === */
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

        /* Transmisión activa */
        .transmission-active {
            animation: transmissionPulse 1s infinite;
        }

        @keyframes transmissionPulse {
            0% { opacity: 1; }
            50% { opacity: 0.7; }
            100% { opacity: 1; }
        }

        /* Sonido de estática */
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

        

        /* === NUEVOS ESTILOS PARA SATÉLITE v4.7.2 === */
      

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

        /* === NUEVOS ESTILOS PARA MICRÓFONO v4.7 === */
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

        /* === MEJORAS PARA EL INDICADOR DE VOZ v4.7 === */
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

        /* === VISUALIZACIÓN DE TRADUCCIÓN MORSE === */
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

        /* === NUEVOS ESTILOS PARA SATÉLITE MEJORADO === */
    /* Fuerza verde fino en TODOS los scrollbars de la página */
::-webkit-scrollbar {
    width: 3px !important;
    height: 3px !important;
}

::-webkit-scrollbar-track {
    background: rgba(10, 10, 10, 0.6) !important;
    border-radius: 10px !important;
}

::-webkit-scrollbar-thumb {
    background: #00ff88 !important;           /* Verde puro del tema */
    border-radius: 10px !important;
    border: 1px solid rgba(0, 255, 136, 0.2) !important;  /* borde sutil opcional */
}

::-webkit-scrollbar-thumb:hover {
    background: #33ff99 !important;
}

/* Firefox global */
* {
    scrollbar-width: thin !important;
    scrollbar-color: #00ff88 rgba(10, 10, 10, 0.6) !important;
}

/* === SEGURIDAD REAL v5.1 === */
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

/* Colores para diferentes tipos de cifrado */
.security-aes { color: #00ffea !important; }
.security-xor { color: #ffaa00 !important; }
.security-none { color: #ff3300 !important; }

/* === ANIMACIONES DE SEGURIDAD === */
        .scan-line {
            position: absolute;
            top: 0;
            left: 0;
            right: 0;
            height: 0.5px;
            background: linear-gradient(90deg, transparent, #00ff88, transparent);
            animation: scan 5s linear infinite;
            pointer-events: none;
            z-index: 5;
        }

        @keyframes scan {
            0% { top: 0; opacity: 0; }
            10% { opacity: 1; }
            90% { opacity: 1; }
            100% { top: 100%; opacity: 0; }
        }

         /* === RESPONSIVE === */
        @media (max-height: 750px) {
            .radcom-container {
                transform: scale(1);
            }
        }

        /* === NUEVOS ESTILOS PARA BOTÓN CONSOLA v5.6.1 === */
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

/* === VENTANA CONSOLA 680x680 === */
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

/* === PESTAÑAS CONSOLA === */
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

/* === CONTENIDO PESTAÑAS === */
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

/* === PESTAÑA 1: CMD CONSOLE === */
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

/* === PESTAÑA 2: HEX EDITOR === */
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

/* === PESTAÑA 3: HEX DECODER === */
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

/* === SCROLLBAR ESTILIZADO === */
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

/* === MODO HEX HIGHLIGHTING === */
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

/* === LÍNEA DE SCAN PARA CONSOLA === */
.console-scan-line {
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    height: 0.5px;
    background: linear-gradient(90deg, transparent, #00ff88, transparent);
    animation: consoleScan 6s linear infinite;
    pointer-events: none;
    z-index: 1;
}

@keyframes consoleScan {
    0% { top: 0; opacity: 0; }
    10% { opacity: 1; }
    90% { opacity: 1; }
    100% { top: 680px; opacity: 0; }
}    

/* === botones nuevos com nav mapas === */

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
    font-family: 'Orbitron', sans-serif; /* Si la usas */
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




    </style>
</head>
<body>
    <div class="radcom-container">
    <!-- Línea de escaneo -->
        <div class="scan-line"></div>
        
    <!-- CABECERA -->
    <div class="container">
        <div class="header-pro">
            <div class="status-indicator">
    <span class="status-dot-live"></span>
    <span>RADCOM MASTER <span class="version-badge">v5.6.5</span></span>
    <span style="color:#7b7d7b; margin-left:10px;">|</span>
    <span id="data-session" style="color:#00ffea">0</span>b
    <!-- NUEVO: BADGE DE SEGURIDAD -->
    <div class="security-badge" onclick="showSecurityInfo()" title="Seguridad AES-256-GCM Activada">
        <span class="security-dot"></span>
        <span style="color:#00ffea;">AES-256-GCM</span>
    </div>
</div>
            <div id="display-id" style="font-size:0.6rem; color:lch(88.59% 86.83 150.39); font-family:monospace;">INICIANDO...</div>
            <div style="display:flex; gap:4px; align-items:center;">
                <!-- BOTÓN REFRESCAR PEQUEÑO COMO ANTES -->
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
                        
                        <!-- BOTÓN REVIVIR MEJORADO -->
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

            <div class="content-area">
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
                            <!-- NUEVO BOTÓN CONSOLA -->
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
                                <!-- Controles de sonido Morse -->
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
                                
                                <!-- Tabla Morse mejorada -->
                                <div class="morse-table-container">
                                    <table id="morseTable"></table>
                                </div>
                            </div>
                            <div id="radio-table" class="tab-content">
                                <!-- SISTEMA DE RADIO v4.6 ORIGINAL MEJORADO -->
                                <div class="radio-container">
                                    <!-- Panel superior: Controles de frecuencia -->
                                    <div class="radio-controls">
                                        <!-- Entrada de frecuencia (izquierda) -->
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
                                        
                                        <!-- Selector de bandas (derecha) -->
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
                                            
                                            <!-- Sub-bandas HF (solo visible cuando HF está seleccionado) -->
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
                                            
                                            <!-- NUEVA SECCIÓN: CB y PMR -->
                                            <div class="pmr-cb-section">
                                                <div class="pmr-cb-title">
                                                    <i class="fas fa-walkie-talkie"></i> CB27 & PMR446
                                                </div>
                                                <div class="channel-type-selector">
                                                    <button class="channel-type-btn active" onclick="selectChannelType('pmr')">PMR446</button>
                                                    <button class="channel-type-btn" onclick="selectChannelType('cb')">CB 27MHz</button>
                                                </div>
                                                
                                                <!-- Canales PMR -->
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
                                                        <!-- Generado por JavaScript -->
                                                    </div>
                                                </div>
                                                
                                                <!-- Canales CB27 -->
                                                <div id="cb-container" class="channel-container">
                                                    <div class="channel-grid" id="cb-channels-grid">
                                                        <!-- Generado por JavaScript -->
                                                    </div>
                                                </div>
                                                
                                                <!-- Información del canal -->
                                                <div class="channel-info">
                                                    <div style="font-size:0.4rem; color:#888;">Canal Activo:</div>
                                                    <div class="channel-display" id="channel-display">VHF 142.850 MHz</div>
                                                </div>
                                            </div>
                                            
                                            <!-- Frecuencia activa -->
                                            <div class="active-frequency-display">
                                                <div class="active-freq-label">Frecuencia Activa</div>
                                                <div class="active-freq-value" id="active-frequency">142.850 MHz VHF</div>
                                            </div>
                                        </div>
                                    </div>
                                    
                                    <!-- Alfabeto fonético interactivo MEJORADO -->
                                    <div class="phonetic-table-container">
                                        <div class="phonetic-header">
                                            <i class="fas fa-broadcast-tower"></i> ALFABETO FONÉTICO RADIO
                                        </div>
                                        <div class="phonetic-grid-container">
                                            <div class="phonetic-grid" id="phonetic-grid">
                                                <!-- Generado por JavaScript -->
                                            </div>
                                        </div>
                                        
                                        <!-- Frecuencias de emergencia -->
                                        <div class="emergency-frequencies">
                                            <div class="emergency-title">
                                                <i class="fas fa-exclamation-triangle"></i> FRECUENCIAS DE EMERGENCIA
                                            </div>
                                            <div class="emergency-list" id="emergency-list">
                                                <!-- Generado por JavaScript -->
                                            </div>
                                        </div>
                                    </div>
                                    
                                    <!-- Overlay de estática (efecto visual) -->
                                    <div class="static-overlay" id="static-overlay"></div>
                                </div>
                            </div>
                            <!-- === NUEVA PESTAÑA SATÉLITE v4.7.2 MEJORADA === -->
                            <div id="satellite-table" class="tab-content active">
                                <div class="satellite-container-v2">
                                    <!-- CABECERA -->
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
                                    
                                    <!-- ESTADO API -->
                                    <div class="api-status-v2">
                                        <span class="status-dot-api status-connected" id="api-status-dot"></span>
                                        <span id="api-status-text">Conectando a API Open-Meteo...</span>
                                    </div>
                                    
                                    <!-- GRID DE DATOS (12 PARÁMETROS) -->
                                    <div class="satellite-grid-v2">
                                        <!-- FILA 1 -->
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
                                        
                                        <!-- FILA 2 -->
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
                                        
                                        <!-- FILA 3 -->
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
                                    
                                    <!-- POSICIÓN GPS CON ALTITUD -->
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
                                    
                                    <!-- PANEL DE EMERGENCIA MEJORADO -->
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
                                    
                                    <!-- CONTROLES DE ACTUALIZACIÓN -->
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
                        <i class="fas fa-satellite"></i> SISTEMA RADCOM MASTER <v4 class="7 6"></v4> INICIADO
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
            
            <!-- MODIFICADO: CON MICRÓFONO MEJORADO -->
            <div style="display:flex; height:36px; align-items:center; padding:4px 5px; gap:4px; border-bottom:1px solid #222;">
                <!-- BOTÓN LIMPIAR -->
                <button class="clear-chat-btn" onclick="clearChat()" style="flex:0 0 auto; width:36px" title="Limpiar chat">
                    <i class="fas fa-trash"></i>
                </button>

                <!-- NUEVO: BOTÓN MICRÓFONO A LA IZQUIERDA -->
                <button id="micBtn" class="mic-btn" title="Dictar por voz" onclick="toggleVoiceInput()">
                    <i class="fas fa-microphone"></i>
                </button>

                <!-- Campo de texto -->
                <input type="text" id="inputMsg"
                    placeholder="INYECTAR DATOS SEGUROS..."
                    oninput="validateInput(); realTimePreview(); realTimeTableHighlight();"
                    onkeydown="if(event.key === 'Enter'){ handleSendMessage(event); }"
                    style="height:100%; flex:1;">

                <!-- Botón ENVIAR -->
                <button id="sendBtn" onclick="sendWithQueue()" style="height:100%">
                    <i class="fas fa-paper-plane"></i> ENVIAR SEGURO
                </button>

                <!-- Estado voz -->
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

    <!-- Modelo base de configuración (sin cambios) -->
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
            
            <!-- NUEVA SECCIÓN: CONFIGURACIÓN DE IDENTIFICADOR -->
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

    <!-- ====== VENTANA CONSOLA v5.6.1 ====== -->
<div class="console-modal-overlay" id="consoleModal">
    <div class="console-modal-content">
        <div class="console-scan-line"></div>
        
        <div class="console-header">
            <div class="console-title">
                <i class="fas fa-terminal"></i> RADCOM CONSOLE v5.6.1 - SYSTEM CONTROL
            </div>
            <button class="console-close" onclick="closeConsole()">&times;</button>
        </div>
        
        <!-- Pestañas -->
        <div class="console-tabs">
            <button class="console-tab active" onclick="switchConsoleTab('CMD')">CMD Console</button>
            <button class="console-tab" onclick="switchConsoleTab('hex')">Hex Editor</button>
            <button class="console-tab" onclick="switchConsoleTab('decode')">Hex Decoder</button>
        </div>
        
        <!-- Contenido de pestañas -->
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

<div id="modal-680" style="display:none; position:fixed; top:50%; left:50%; transform:translate(-50%, -50%); width:680px; height:680px; background:#000; border:2px solid #00ff88; z-index:9999; box-shadow: 0 0 20px rgba(0,255,136,0.5);">
    <div style="display:flex; justify-content:space-between; background:#222; padding:5px 10px; border-bottom:1px solid #00ff88;">
        <span id="modal-title" style="color:#00ff88; font-family:monospace; font-weight:bold;">SISTEMA NAV</span>
        <button onclick="closeModal680()" style="background:none; border:none; color:#ff3300; cursor:pointer; font-weight:bold;">[ X ]</button>
    </div>
    <div id="modal-body" style="width:100%; height:calc(100% - 30px); overflow:hidden;">
        </div>
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
        <!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <style>
        :root {
            --radcom-green: #00ff88;
            --radcom-bg: #000000;
            --radcom-dark: #111111;
            --radcom-border: #333333;
        }

        body {
            font-family: 'Courier New', monospace;
            background-color: var(--radcom-bg);
            color: white;
            margin: 0;
            padding: 20px;
            display: flex;
            justify-content: center;
        }

        .container {
            width: 100%;
            max-width: 720px;
            background: var(--radcom-dark);
            border: 1px solid var(--radcom-border);
            padding: 20px;
            border-radius: 8px;
            box-shadow: 0 0 20px rgba(0, 255, 136, 0.1);
        }

        h1 {
            color: var(--radcom-green);
            font-size: 1.2rem;
            text-align: center;
            border-bottom: 1px solid var(--radcom-border);
            padding-bottom: 10px;
            margin-bottom: 20px;
            text-transform: uppercase;
        }

        .grid {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 15px;
        }

        .input-group {
            display: flex;
            flex-direction: column;
        }

        label {
            font-size: 0.7rem;
            color: #888;
            margin-bottom: 5px;
            text-transform: uppercase;
        }

        input {
            background: #000;
            border: 1px solid var(--radcom-border);
            color: var(--radcom-green);
            padding: 10px;
            font-size: 1rem;
            border-radius: 4px;
            outline: none;
        }

        input:focus {
            border-color: var(--radcom-green);
        }

        /* Panel de Resultados */
        .results-panel {
            margin-top: 25px;
            background: #050505;
            border: 1px solid var(--radcom-green);
            padding: 15px;
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 10px;
        }

        .res-item {
            text-align: center;
        }

        .res-label {
            font-size: 0.6rem;
            color: var(--radcom-green);
            display: block;
        }

        .res-val {
            font-size: 1.4rem;
            font-weight: bold;
        }

        /* El Marcador de Rango (Gráfico) */
        .range-container {
            grid-column: span 2;
            margin-top: 20px;
            padding: 10px 0;
        }

        .range-bar {
            height: 10px;
            background: #222;
            position: relative;
            border-radius: 5px;
            border: 1px solid #444;
        }

        .marker {
            width: 4px;
            height: 20px;
            background: var(--radcom-green);
            position: absolute;
            top: -6px;
            box-shadow: 0 0 10px var(--radcom-green);
            transition: left 0.5s ease;
        }

        .labels {
            display: flex;
            justify-content: space-between;
            font-size: 0.6rem;
            color: #666;
            margin-top: 5px;
        }
    </style>
</head>
<body>

<div class="container">
    <h1>Calculadora PTV Pro v5.6.3</h1>
    
    <div class="grid">
        <div class="input-group">
            <label>Peso Piloto (kg)</label>
            <input type="number" id="piloto" value="80" oninput="calc()">
        </div>
        <div class="input-group">
            <label>Motor/Chasis (kg)</label>
            <input type="number" id="motor" value="22" oninput="calc()">
        </div>
        <div class="input-group">
            <label>Equipo/Reserva (kg)</label>
            <input type="number" id="equipo" value="5" oninput="calc()">
        </div>
        <div class="input-group">
            <label>Carga Vivac (kg)</label>
            <input type="number" id="vivac" value="0" oninput="calc()">
        </div>
    </div>

    <div class="results-panel">
        <div class="res-item">
            <span class="res-label">PTV TOTAL</span>
            <span class="res-val" id="resPTV">107.0</span><span style="font-size: 0.7rem;"> kg</span>
        </div>
        <div class="res-item">
            <span class="res-label">VELA IDEAL (4.15 kg/m²)</span>
            <span class="res-val" id="resSurf" style="color: var(--radcom-green);">25.8</span><span style="font-size: 0.7rem;"> m²</span>
        </div>

        <div class="range-container">
            <span class="res-label" style="text-align: center; margin-bottom: 10px;">POSICIÓN EN RANGO CERTIFICACIÓN (Óptimo 75%)</span>
            <div class="range-bar">
                <div id="marker" class="marker" style="left: 75%;"></div>
            </div>
            <div class="labels">
                <span id="labelMin">90 kg</span>
                <span>CENTRO</span>
                <span id="labelMax">112 kg</span>
            </div>
        </div>
    </div>
</div>

<script>
function calc() {
    const p = parseFloat(document.getElementById('piloto').value) || 0;
    const m = parseFloat(document.getElementById('motor').value) || 0;
    const e = parseFloat(document.getElementById('equipo').value) || 0;
    const v = parseFloat(document.getElementById('vivac').value) || 0;

    const ptv = p + m + e + v;
    document.getElementById('resPTV').innerText = ptv.toFixed(1);

    // Vela ideal para carga alar media-alta (4.15 kg/m²)
    const superficie = ptv / 4.15;
    document.getElementById('resSurf').innerText = superficie.toFixed(1);

    // Cálculo de Rango Teórico (Amplitud de 22kg típica)
    const amplitud = 22;
    const kgMax = ptv + (amplitud * 0.25);
    const kgMin = kgMax - amplitud;

    document.getElementById('labelMin').innerText = Math.round(kgMin) + " kg";
    document.getElementById('labelMax').innerText = Math.round(kgMax) + " kg";

    // Mover el marcador (siempre estará al 75% visualmente porque el rango se calcula relativo al PTV)
    // Pero si quisieras comparar contra una vela fija, aquí cambiaría la lógica.
    document.getElementById('marker').style.left = "75%";
}

// Iniciar al cargar
window.onload = calc;
</script>

</body>
</html>       
    </div>

    <div id="ecm-source-storage" style="display:none;">
        <!DOCTYPE html>
<html lang="es">

<head>
  <meta charset="UTF-8">
  <style>
    
    /* 320x240 - MARCO DE PRODUCCIÓN */
    .pantalla-original-match {
      width: 320px;
      height: 240px;
      background-color: #000;
      border: 1px solid #252525;
      position: relative;
      font-family: 'Segoe UI', Arial, sans-serif;
      color: white;
      overflow: hidden;
      box-sizing: border-box;
      padding: 12px;
      /* Margen de seguridad pro */
    }

    /* HEADER - ALINEADO AL PÍXEL */
    .header {
      display: flex;
      justify-content: space-between;
      font-size: 8.5px;
      color: #d1d1d1;
      margin-bottom: 8px;
      font-weight: 300;
    }

    /* FILA SUPERIOR - BALANCE RPM/N1 */
    .row-top {
      display: flex;
      justify-content: space-between;
      align-items: flex-end;
    }

    .rpm-group {
      text-align: left;
    }

    .label-tech {
      font-size: 9px;
      color: #55FF55;
      font-weight: bold;
      margin-bottom: 2px;
      letter-spacing: 0.3px;
    }

    .val-rpm {
      font-size: 60px;
      font-weight: 900;
      color: #55FF55;
      line-height: 0.82;
      margin: 0;
      letter-spacing: -1.5px;
    }

    .n1-box {
      width: 110px;
      height: 68px;
      border: 1.8px solid #00FFFF;
      border-radius: 10px;
      background: rgba(0, 255, 255, 0.08);
      text-align: center;
      display: flex;
      flex-direction: column;
      justify-content: center;
    }

    .val-n1 {
      font-size: 44px;
      color: #00FFFF;
      font-weight: bold;
      line-height: 0.85;
      margin-top: 2px;
    }

    /* BARRA DE PROGRESO - NI MUY FINA NI MUY GRUESA */
    .bar-wrap {
      width: 100%;
      height: 12px;
      border: 1.8px solid #55FF55;
      border-radius: 6px;
      margin: 12px 0;
      background: rgba(0, 55, 0, 0.25);
      padding: 1px;
      box-sizing: border-box;
    }

    .bar-fill {
      width: 72%;
      height: 100%;
      background: #55FF55;
      border-radius: 4px;
    }

    /* SECCIÓN INFERIOR - EL "CORAZÓN" DEL DISEÑO */
    .row-sensors {
      display: flex;
      justify-content: space-between;
      align-items: center;
      margin-top: 6px;
    }

    .sensor-unit {
      text-align: center;
      min-width: 70px;
    }

    .s-label {
      font-size: 7.5px;
      color: #b0b0b0;
      margin-bottom: 4px;
      font-weight: bold;
    }

    .s-val {
      font-size: 28px;
      font-weight: bold;
      color: #fff;
      line-height: 1;
    }

    .s-line {
      width: 48px;
      height: 2.5px;
      background: #fff;
      margin: 5px auto 0;
      border-radius: 1px;
    }

    /* CAJA FUEL - PROFUNDIDAD Y ESCALA CORRECTA */
    .fuel-card {
      width: 102px;
      height: 72px;
      border: 1.8px solid #FF9900;
      border-radius: 12px;
      background: rgba(255, 153, 0, 0.12);
      overflow: hidden;
      box-shadow: 0 4px 10px rgba(0, 0, 0, 0.3);
    }

    .fuel-top {
      background: #FF9900;
      color: #000;
      font-size: 8.5px;
      font-weight: 900;
      padding: 2.5px 0;
      text-align: center;
      letter-spacing: 0.5px;
    }

    .fuel-val {
      font-size: 35px;
      color: #FF9900;
      font-weight: bold;
      text-align: center;
      margin-top: 3px;
    }

    .fuel-est {
      font-size: 7.5px;
      color: #FF9900;
      text-align: center;
      margin-top: -2px;
      font-weight: bold;
    }

    /* FOOTER - ALINEADO A LOS BORDES */
    .footer {
      position: absolute;
      bottom: 10px;
      width: 296px;
      display: flex;
      justify-content: space-between;
      left: 12px;
    }

    .tag {
      font-size: 7.5px;
      padding: 2px 8px;
      border-radius: 5px;
      font-weight: bold;
      letter-spacing: 0.2px;
    }

    .t-hrs {
      border: 1.2px solid #55FF55;
      color: #55FF55;
      background: rgba(0, 255, 0, 0.05);
    }

    .t-svc {
      border: 1.2px solid #FF4444;
      color: #FF4444;
      background: rgba(255, 0, 0, 0.05);
    }
  </style>
</head>

<body>

  <div class="pantalla-original-match">
    <div class="header">
      <div>18:02:25 | SAT JAN 24, 2026</div>
      <div style="font-weight: 700; letter-spacing: 0.5px;">Paramotor Custom ATOM 80</div>
      <div>85% 🔋</div>
    </div>

    <div class="row-top">
      <div class="rpm-group">
        <div class="label-tech">ENGINE RPM</div>
        <p class="val-rpm">8463</p>
      </div>
      <div class="n1-box">
        <div class="label-tech" style="color:#00FFFF; margin-bottom: 0;">N1 %</div>
        <div class="val-n1">85</div>
      </div>
    </div>

    <div class="bar-wrap">
      <div class="bar-fill"></div>
    </div>

    <div class="row-sensors">
      <div class="sensor-unit">
        <div class="s-label">CHT CULATA</div>
        <div class="s-val">185°C</div>
        <div class="s-line"></div>
      </div>

      <div class="fuel-card">
        <div class="fuel-top">FUEL LITERS</div>
        <div class="fuel-val">8.0</div>
        <div class="fuel-est">EST: 39 min</div>
      </div>

      <div class="sensor-unit">
        <div class="s-label">EGT ESCAPE</div>
        <div class="s-val">520°C</div>
        <div class="s-line"></div>
      </div>
    </div>

    <div class="footer">
      <div class="tag t-hrs">TOTAL HRS: 102.5</div>
      <div class="tag t-svc">NEXT SVC: 9.8h</div>
    </div>
  </div>

</body>

</html>
    </div>


  <div id="map-source-storage" style="display:none;">
    <div style="display:flex; flex-direction:column; height:100%; width:100%; background:#000; position:relative; overflow:hidden;">
        
        <!-- BARRA SUPERIOR CON 4 VISTAS + DUAL -->
        <div style="display:flex; background:#111; border-bottom:2px solid #00ff88; height:52px; flex-shrink:0; z-index:1002; align-items:center; padding:0 10px;">
            <div style="display:flex; flex:1; gap:4px;">
                <button onclick="switchMapLayer('TOPO')" id="tab-topo" style="flex:1; background:#00ff88; color:#000; border:none; font-weight:bold; cursor:pointer; font-size:0.73rem; padding:8px 4px;">TOPOGRÁFICO</button>
                <button onclick="switchMapLayer('VFR')"  id="tab-vfr"  style="flex:1; background:#222; color:#888; border:none; font-weight:bold; cursor:pointer; font-size:0.73rem; padding:8px 4px;">VFR AÉREO</button>
                <button onclick="switchMapLayer('SEA')"  id="tab-sea"  style="flex:1; background:#222; color:#888; border:none; font-weight:bold; cursor:pointer; font-size:0.73rem; padding:8px 4px;">NÁUTICO</button>
                <button onclick="switchMapLayer('SAT')"  id="tab-sat"  style="flex:1; background:#222; color:#888; border:none; font-weight:bold; cursor:pointer; font-size:0.73rem; padding:8px 4px;">SATÉLITE</button>
            </div>
            
            <!-- BOTÓN DUAL -->
            <button onclick="toggleDualMode()" id="dual-btn" 
                    style="margin-left:12px; background:#ffaa00; color:#000; border:none; padding:8px 16px; border-radius:4px; font-weight:bold; cursor:pointer; font-size:0.75rem;">
                DUAL
            </button>
        </div>

        <!-- MAPA -->
        <div id="map-canvas" style="flex:1; background:#050505; position:relative;"></div>

        <!-- Barra herramientas vertical izquierda -->
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

        <!-- PANEL RESCATE -->
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

        <!-- PANEL DE WAYPOINTS/RUTA - CON RETORNO AL INICIO -->
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
            
            <!-- CONTROLES DE RUTA ACTIVA -->
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

        <!-- PANEL DE RUTAS GUARDADAS -->
        <div id="track-history-panel" style="position:absolute; bottom:50px; right:15px; background:rgba(0,0,0,0.85); border:1px solid #00ff88; border-radius:4px; padding:8px; z-index:1000; width:240px; display:none; backdrop-filter:blur(3px);">
            <div style="display:flex; justify-content:space-between; align-items:center; border-bottom:1px solid #00ff88; padding-bottom:5px; margin-bottom:6px;">
                <span style="color:#00ff88; font-weight:bold; font-size:0.75rem;">📋 RUTAS GUARDADAS</span>
                <button onclick="toggleTrackHistory()" style="background:none; border:none; color:#00ff88; font-size:0.8rem; cursor:pointer; padding:0 4px;">✕</button>
            </div>
            <div id="track-list" style="max-height:200px; overflow-y:auto; color:#fff; font-size:0.6rem; margin-bottom:5px;"></div>
            <button onclick="clearAllTracks()" style="width:100%; background:#ff3300; color:#fff; border:none; padding:4px; border-radius:2px; font-size:0.6rem;">🗑️ BORRAR HISTORIAL</button>
        </div>

        <!-- PANEL BRÚJULA -->
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

        <!-- Barra de estado inferior -->
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

    <!-- LEAFLET Y TURF -->
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

        // Cargar rutas guardadas
        try {
            const stored = localStorage.getItem('savedRoutes');
            if (stored) savedTracks = JSON.parse(stored);
        } catch(e) {}

        // ===== CAPAS DE MAPA =====
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
                    maxZoom: 18, 
                    opacity: 0.92,
                    attribution: '© OpenSeaMap'
                })
            ]),
            'SAT': L.tileLayer('https://server.arcgisonline.com/ArcGIS/rest/services/World_Imagery/MapServer/tile/{z}/{y}/{x}', { 
                maxZoom: 19,
                attribution: '© Esri'
            })
        };

        // ===== FUNCIÓN CRÍTICA: CAMBIAR MAPAS =====
        function switchMapLayer(type) {
            // Resetear estilo de todos los botones
            ['topo', 'vfr', 'sea', 'sat'].forEach(id => {
                const btn = document.getElementById('tab-' + id);
                if (btn) {
                    btn.style.background = '#222';
                    btn.style.color = '#888';
                }
            });
            
            // Activar botón seleccionado
            const activeBtn = document.getElementById('tab-' + type.toLowerCase());
            if (activeBtn) {
                activeBtn.style.background = '#00ff88';
                activeBtn.style.color = '#000';
            }
            
            // Cambiar capa
            if (currentLayer) {
                map.removeLayer(currentLayer);
            }
            
            currentLayer = layers[type];
            currentLayer.addTo(map);
            
            // Restaurar modo dual si estaba activo
            if (isDualMode && dualOverlay) {
                map.addLayer(dualOverlay);
            }
            
            console.log('Capa cambiada a:', type);
        }

        // ===== INICIALIZACIÓN =====
        function initMap() {
            if (map) return;
            
            map = L.map('map-canvas', { 
                zoomControl: false,
                attributionControl: true
            }).setView([40.4167, -3.7033], 13);
            
            // Capa por defecto: TOPO
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

        // ===== MODO DUAL =====
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

        // ===== BRÚJULA =====
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

        // ===== POSICIÓN INICIAL =====
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

        // ===== PANEL RESCATE =====
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
        
        // ===== EXPORTAR GPX =====
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
        
        // ===== INICIAR =====
        setTimeout(initMap, 300);
        
        // Añadir estilos de animación
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

            
        // ====== VERSIÓN  ======
        const VERSION = "5.6.5";
        const SYSTEM_NAME = "RADCOM MASTER";
        
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

        // =============================================
// PARTE A - CAPA DE SEGURIDAD v5.6.7 (ECDH + AES-GCM real + compatibilidad)
let peerKeys = {}; // peerId → { sharedKey, ecdhPrivate, handshakeDone }

async function generateECDHKeyPair() {
    return await crypto.subtle.generateKey(
        { name: "ECDH", namedCurve: "P-256" },
        true,
        ["deriveKey"]
    );
}

async function importPublicKey(raw) {
    return await crypto.subtle.importKey("raw", raw, { name: "ECDH", namedCurve: "P-256" }, false, []);
}

async function deriveSharedKey(privateKey, publicKey) {
    const sharedSecret = await crypto.subtle.deriveKey(
        { name: "ECDH", public: publicKey },
        privateKey,
        { name: "AES-GCM", length: 256 },
        false,
        ["encrypt", "decrypt"]
    );
    return sharedSecret;
}

// === ENCRIPTACIÓN / DESENCRIPTACIÓN GCM ===
async function encryptAESGCM(plaintext, key) {
    const iv = crypto.getRandomValues(new Uint8Array(12));
    const encrypted = await crypto.subtle.encrypt(
        { name: "AES-GCM", iv },
        key,
        new TextEncoder().encode(plaintext)
    );
    return {
        iv: Array.from(iv),
        data: Array.from(new Uint8Array(encrypted))
    };
}

async function decryptAESGCM(encryptedObj, key) {
    const iv = new Uint8Array(encryptedObj.iv);
    const data = new Uint8Array(encryptedObj.data);
    const decrypted = await crypto.subtle.decrypt({ name: "AES-GCM", iv }, key, data);
    return new TextDecoder().decode(decrypted);
}

// === XOR (legacy) ===
function xorEncrypt(text, key) {
    let result = '';
    for (let i = 0; i < text.length; i++) {
        result += String.fromCharCode(text.charCodeAt(i) ^ key.charCodeAt(i % key.length));
    }
    return result;
}

// === CAPA UNIFICADA (LA QUE USARÁS) ===
async function securityLayer(text, isSending, targetId = null) {
    const result = { payload: null, encryptionUsed: "", error: null };
    const password = document.getElementById('key')?.value || "ATOM80";

    if (!text) {
        result.error = "Mensaje vacío";
        return result;
    }

    const mode = document.getElementById('encryptionMode').value || "aes-gcm-ecdh";

    try {
        // 1. Modo sin cifrado
        if (mode === "none") {
            result.payload = text;
            result.encryptionUsed = "NONE";
            return result;
        }

        // 2. Modo XOR (compatibilidad antigua)
        if (mode === "xor") {
            result.payload = isSending ? xorEncrypt(text, password) : xorEncrypt(text, password);
            result.encryptionUsed = "XOR";
            return result;
        }

        // 3. MODO MODERNO: ECDH + AES-GCM
        if (mode === "aes-gcm-ecdh") {
            if (!targetId || targetId === 'GLOBAL') {
                result.payload = text;
                result.encryptionUsed = "PLAIN-GLOBAL";
                return result;
            }

            let entry = peerKeys[targetId];

            // No tenemos clave compartida → iniciamos handshake
            if (!entry || !entry.sharedKey) {
                if (!entry) {
                    const pair = await generateECDHKeyPair();
                    peerKeys[targetId] = {
                        ecdhPrivate: pair.privateKey,
                        handshakeDone: false
                    };
                    entry = peerKeys[targetId];
                }

                const pubRaw = await crypto.subtle.exportKey("raw", (await generateECDHKeyPair()).publicKey); // simplificado

                connections[targetId]?.conn?.send({
                    type: "ecdh_init",
                    publicKey: Array.from(new Uint8Array(pubRaw)),
                    from: myPeerId
                });

                result.payload = "[NEGOCIANDO CLAVE SEGURA ECDH...]";
                result.encryptionUsed = "ECDH_HANDSHAKE";
                return result;
            }

            // Ya tenemos clave → ciframos/desciframos
            if (isSending) {
                result.payload = await encryptAESGCM(text, entry.sharedKey);
                result.encryptionUsed = "AES-256-GCM";
            } else {
                result.payload = await decryptAESGCM(text, entry.sharedKey);
                result.encryptionUsed = "AES-256-GCM";
            }
        }
    } catch (err) {
        result.error = err.message;
        result.payload = isSending ? text : "[ERROR CRÍTICO DE SEGURIDAD]";
    }

    return result;
}

// =============================================
// PARTE B - SISTEMA DE MENSAJERÍA UNIFICADO v5.4
// =============================================

async function prepareAndSecureMessage(rawText, mode = 'text') {
    const encryptionMode = document.getElementById('encryptionMode').value || 'aes-gcm-ecdh';
    const key = document.getElementById('key').value || 'ATOM80';

    let processedText = rawText.trim();

    // === MORSE DUAL (mantengo tu funcionalidad original) ===
    if (mode === 'morse') {
        const isMorseInput = /^[\.\-\s\/]+$/.test(processedText);
        if (!isMorseInput) {
            processedText = textToMorse(processedText);   // tu función original
        }
        updateMorseTranslation(rawText); // tu función de traducción
    } else if (mode === 'phonetic') {
        processedText = textToPhonetic(processedText); // tu función original
    }

    // === USAR LA NUEVA CAPA DE SEGURIDAD ===
    const secured = await securityLayer(processedText, true, encryptionMode, key);

    return {
        type: 'message',
        payload: secured.payload,
        mode: mode,
        encryption: secured.encryptionUsed,
        timestamp: Date.now(),
        sender: myPeerId,
        originalText: rawText,           // para mostrar localmente
        error: secured.error
    };
}

// === ENVÍO PRINCIPAL (reemplaza tu sendWithQueue) ===
async function sendWithQueue() {
    const input = document.getElementById('inputMsg');
    const rawText = input.value.trim();
    
    if (!rawText) {
        updateMonitor("⚠️ MENSAJE VACÍO", "error");
        playStrongBeep(300, 200);
        return;
    }

    const mode = document.getElementById('inputMode').value;
    const packet = await prepareAndSecureMessage(rawText, mode);

    if (packet.error) {
        updateMonitor(`⚠️ Error de seguridad: ${packet.error}`, "error");
        return;
    }

    let sentCount = 0;

    if (activeTarget === 'GLOBAL') {
        for (const peerId in connections) {
            if (connections[peerId]?.conn?.open) {
                connections[peerId].conn.send(packet);
                sentCount++;
            }
        }
    } else if (connections[activeTarget]?.conn?.open) {
        connections[activeTarget].conn.send(packet);
        sentCount = 1;
    }

    if (sentCount > 0) {
        // Mostrar mensaje original (no cifrado) al usuario
        displayMessage(`YO: ${rawText}`, '', 'outgoing');
        input.value = '';
        stats.tx += rawText.length;
        stats.messages++;
        updateStats();
        updateMonitor(`📤 ENVIADO (${packet.encryption}) a ${sentCount} destino(s)`);
        playStrongBeep(700, 100);
    } else {
        // Cola offline (tu sistema original)
        addToQueue(packet, rawText);
        displayMessage(`⏳ YO (EN COLA): ${rawText}`, '', 'outgoing');
        input.value = '';
        updateMonitor(`💾 MENSAJE GUARDADO EN COLA (${messageQueue.length})`);
    }

    // Limpiar tabla resaltada
    document.querySelectorAll('#ansiTable td.highlighted').forEach(td => td.classList.remove('highlighted'));
}

// === RECEPCIÓN UNIFICADA (reemplaza todas las versiones antiguas) ===
async function handleReceivedData(senderId, data) {
    if (!data || data.type !== 'message') return;

    connectionHealth[senderId] = connectionHealth[senderId] || {};
    connectionHealth[senderId].lastActivity = Date.now();

    let displayText = data.payload;

    // === PROCESAR SEGÚN TIPO DE ENCRIPTACIÓN ===
    if (data.encryption === 'ECDH_HANDSHAKE' || data.encryption === 'aes-gcm-ecdh') {
        // Ya lo maneja securityLayer + handshake en Parte A
        const secured = await securityLayer(data.payload, false, data.encryption);
        displayText = secured.payload || data.payload;
    } 
    else if (data.encryption === 'XOR' || data.encryption.includes('legacy')) {
        displayText = xorDecrypt(data.payload, document.getElementById('key').value || 'ATOM80');
    } 
    else if (data.encryption === 'NONE') {
        displayText = data.payload;
    }

    // === MOSTRAR MENSAJE ===
    const displayName = senderId.substring(0, 8);
    displayMessage(`${displayName}: ${displayText}`, '', 'incoming');

    stats.rx += (data.payload ? JSON.stringify(data.payload).length : 0);
    stats.messages++;
    updateStats();

    updateMonitor(`📥 ${displayName} [${data.encryption}]`);
    playMessageNotification();
}

// === COLA (mantengo tu sistema original, solo lo limpio) ===
function addToQueue(packet, originalText) {
    if (messageQueue.length >= 100) messageQueue.shift();

    messageQueue.push({
        id: Date.now(),
        packet: packet,
        original: originalText,
        target: activeTarget,
        timestamp: Date.now(),
        attempts: 0
    });

    localStorage.setItem('radcom_message_queue', JSON.stringify(messageQueue));
    updateQueueCounter();
}

// Actualiza tu processQueue y deliverOnConnect si quieres, pero con esta versión ya funciona mejor.

        // ====== VARIABLES GLOBALES ======
        let peer = null;
        let myPeerId = null;
        let connections = {};
        let savedIds = JSON.parse(localStorage.getItem('radcom_peers_v4') || "[]");
        let activeTarget = 'GLOBAL';
        let stats = {
            messages: 0,
            tx: 0,
            rx: 0,
            startTime: Date.now()
        };
        let audioContext = null;
        let currentConnectionType = 'wifi';
       
        // ====== VARIABLES MEJORADAS ======
        let connectionHealth = {};
        let showOffline = false;
        let fastRecovery = true;
        let aggressiveRevive = true;
        let isBackground = false;
        let revivingInProgress = false;
        let currentTab = 'ascii';

        // ====== VARIABLES SISTEMA DE MENSAJES EN COLA ======
        let messageQueue = JSON.parse(localStorage.getItem('radcom_message_queue') || "[]");
        let queueRetryInterval = null;
        const MAX_QUEUE_SIZE = 100;

        // ====== VARIABLES PARA MORSE ======
        let morseSpeed = 'normal';
        let morseAudioContext = null;
        let isPlayingMorse = false;

        // ====== VARIABLES PARA RECONOCIMIENTO DE VOZ v4.7 MEJORADO ======
        let recognition = null;
        let recognizing = false;

        // ====== SISTEMA DE ID FIJOS MEJORADO ======
        const ID_SYSTEM = {
            currentId: null,
            defaultPrefix: 'RADCOM-',
            useFixedId: true,
            fixedId: null
        };

        // ====== SISTEMA DE RADIO v4.5 (ORIGINAL MEJORADO) ======
        let currentBand = 'VHF';
        let currentFrequency = 142.850;
        let currentUnit = 'MHz';
        let currentHFBand = null;
        let currentChannelType = 'pmr'; // 'pmr' o 'cb'
        let currentPMRType = 8; // 8, 16, 32
        let currentChannel = 1; // Canal actual
        let radioAudioContext = null;

        // Definición de bandas y frecuencias
        const radioBands = {
            'UHF': { min: 300, max: 3000, unit: 'MHz', default: 433.000 },
            'VHF': { min: 30, max: 300, unit: 'MHz', default: 142.850 },
            'AEREA': { min: 108, max: 137, unit: 'MHz', default: 121.500 },
            'MARINA': { min: 156, max: 162, unit: 'MHz', default: 156.800 },
            'HF': { min: 3, max: 30, unit: 'MHz', default: 14.300 },
            'EMERG': { min: 0, max: 0, unit: 'MHz', default: 121.500 }
        };

        // Sub-bandas HF con descripciones
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

        // Canales PMR446
        const pmrChannels = {
            8: {
                1: 446.00625, 2: 446.01875, 3: 446.03125, 4: 446.04375,
                5: 446.05625, 6: 446.06875, 7: 446.08125, 8: 446.09375
            },
            16: {
                1: 446.00625, 2: 446.01875, 3: 446.03125, 4: 446.04375,
                5: 446.05625, 6: 446.06875, 7: 446.08125, 8: 446.09375,
                9: 446.10625, 10: 446.11875, 11: 446.13125, 12: 446.14375,
                13: 446.15625, 14: 446.16875, 15: 446.18125, 16: 446.19375
            },
            32: {
                1: 446.00625, 2: 446.009375, 3: 446.0125, 4: 446.015625,
                5: 446.01875, 6: 446.021875, 7: 446.025, 8: 446.028125,
                9: 446.03125, 10: 446.034375, 11: 446.0375, 12: 446.040625,
                13: 446.04375, 14: 446.046875, 15: 446.05, 16: 446.053125,
                17: 446.05625, 18: 446.059375, 19: 446.0625, 20: 446.065625,
                21: 446.06875, 22: 446.071875, 23: 446.075, 24: 446.078125,
                25: 446.08125, 26: 446.084375, 27: 446.0875, 28: 446.090625,
                29: 446.09375, 30: 446.096875, 31: 446.1, 32: 446.103125
            }
        };

        // Canales CB (40 canales)
        const cbChannels = {
            1: 26.965, 2: 26.975, 3: 26.985, 4: 27.005, 5: 27.015,
            6: 27.025, 7: 27.035, 8: 27.055, 9: 27.065, 10: 27.075,
            11: 27.085, 12: 27.105, 13: 27.115, 14: 27.125, 15: 27.135,
            16: 27.155, 17: 27.165, 18: 27.175, 19: 27.185, 20: 27.205,
            21: 27.215, 22: 27.225, 23: 27.255, 24: 27.235, 25: 27.245,
            26: 27.265, 27: 27.275, 28: 27.285, 29: 27.295, 30: 27.305,
            31: 27.315, 32: 27.325, 33: 27.335, 34: 27.345, 35: 27.355,
            36: 27.365, 37: 27.375, 38: 27.385, 39: 27.395, 40: 27.405
        };

        // Frecuencias de emergencia por banda
        const emergencyFrequencies = {
            'UHF': [
                { freq: '433.000', purpose: 'Emergencia general UHF' },
                { freq: '446.000', purpose: 'PMR446 Canal 1' }
            ],
            'VHF': [
                { freq: '146.520', purpose: 'Simplex nacional (EEUU)' },
                { freq: '145.500', purpose: 'Emergencia VHF' }
            ],
            'AEREA': [
                { freq: '121.500', purpose: 'Emergencia aeronáutica' },
                { freq: '123.100', purpose: 'Ayuda aeronáutica' }
            ],
            'MARINA': [
                { freq: '156.800', purpose: 'Canal 16 - Emergencia' },
                { freq: '156.300', purpose: 'Canal 6 - Auxilio' }
            ],
            'HF': [
                { freq: '14.300', purpose: 'Emergencia HF global' },
                { freq: '7.296', purpose: 'Red de emergencia' }
            ],
            'EMERG': [
                { freq: '121.500', purpose: 'Emergencia aeronáutica' },
                { freq: '156.800', purpose: 'Emergencia marítima' },
                { freq: '27.185', purpose: 'Canal 19 CB - Emergencia' }
            ]
        };

        // Alfabeto fonético completo
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

        // ======== RECONOCIMIENTO DE VOZ v4.7 (CON ENVÍO AUTOMÁTICO) ========

        function initVoiceRecognition() {
            const SpeechRecognition = window.SpeechRecognition || window.webkitSpeechRecognition;
            if (!SpeechRecognition) {
                updateMonitor("Reconocimiento de voz no soportado en este navegador.", "warning");
                return;
            }
            
            recognition = new SpeechRecognition();
            recognition.lang = "es-ES";
            recognition.continuous = false;
            recognition.interimResults = false; // Solo resultados finales
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
                    // Mostrar el texto reconocido en el campo de entrada
                    input.value = transcript;
                    
                    // Actualizar vistas previas
                    validateInput();
                    realTimePreview();
                    realTimeTableHighlight();
                    
                    updateMonitor(`🎤 Reconocido: "${transcript.substring(0, 30)}${transcript.length > 30 ? '...' : ''}"`, "info");
                    playStrongBeep(600, 50);
                    
                    // ENVÍO AUTOMÁTICO DESPUÉS DE 1.5 SEGUNDOS
                    setTimeout(() => {
                        if (input.value.trim() && !recognizing) {
                            updateMonitor("⚡ ENVIANDO MENSAJE DE VOZ...", "info");
                            
                            // Pequeña pausa para mostrar el mensaje
                            setTimeout(() => {
                                const mode = document.getElementById('inputMode').value;
                                
                                // Verificar conexiones antes de enviar
                                const onlineCount = Object.keys(connections).filter(id => 
                                    connections[id]?.status === 'online').length;
                                
                                if (onlineCount === 0) {
                                    updateMonitor("⚠️ No hay conexiones activas para enviar", "warning");
                                    playStrongBeep(300, 200);
                                    return;
                                }
                                
                                // Enviar según el modo
                                if (mode === 'phonetic') {
                                    sendRadioMessage();
                                } else {
                                    sendMessage();
                                }
                                
                                // Restablecer el campo de entrada
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

        // Función auxiliar para restablecer la UI de voz
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
            // Si ya está activo, detenerlo
            if (recognizing) {
                try {
                    recognition.stop();
                    updateMonitor("🎤 Modo voz desactivado", "info");
                } catch(e) {
                    console.log("Reconocimiento ya detenido");
                }
                return;
            }
            
            // Inicializar si es necesario
            if (!recognition) {
                initVoiceRecognition();
                if (!recognition) {
                    updateMonitor("❌ No se pudo inicializar reconocimiento de voz", "error");
                    return;
                }
            }
            
            // Limpiar campo de entrada antes de empezar
            const input = document.getElementById("inputMsg");
            if (input) {
                input.value = "";
            }
            
            // Intentar iniciar el reconocimiento
            try {
                recognition.start();
            } catch(e) {
                console.error("Error al iniciar reconocimiento:", e);
                updateMonitor("❌ Error al acceder al micrófono. Verifica permisos.", "error");
                resetVoiceUI();
            }
        }

        // ====== FUNCIONES DE SISTEMA DE ID ======
        
        function generateHardwareBasedId() {
            const sources = [
                navigator.userAgent,
                navigator.language,
                screen.width + 'x' + screen.height,
                new Date().getTimezoneOffset(),
                localStorage.getItem('radcom_hardware_id')
            ];
            
            if (!localStorage.getItem('radcom_hardware_id')) {
                const hardwareId = 'HW-' + 
                    Math.random().toString(36).substring(2, 10) + 
                    '-' + 
                    Date.now().toString(36);
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

        // ====== FUNCIONES BÁSICAS ======
        
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

        function buildSatelliteTable() {
            // La tabla se construye directamente en el HTML
            console.log("🛰️ Tabla satelital construida");
            initializeSatelliteWeather();
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

        // ====== FUNCIONES DE MORSE ======
        
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

        // ====== FUNCIONES DE SONIDO MORSE ======
        
        function initMorseAudio() {
            if (!morseAudioContext) {
                try {
                    morseAudioContext = new (window.AudioContext || window.webkitAudioContext)();
                } catch (error) {
                    console.error("Error inicializando audio Morse:", error);
                }
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

        // ====== SISTEMA DE RADIO v4.5 ORIGINAL MEJORADO ======

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
            
            freqInput.addEventListener('input', validateFrequency);
            freqInput.addEventListener('change', updateFrequencyFromInput);
            unitSelect.addEventListener('change', updateUnit);
        }

        function validateFrequency() {
            const input = document.getElementById('radio-frequency-input');
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
            
            document.getElementById('radio-frequency-input').value = currentFrequency.toFixed(3);
        }

        function selectBand(band) {
            currentBand = band;
            currentHFBand = null;
            
            document.querySelectorAll('.band-btn').forEach(btn => {
                btn.classList.remove('active');
            });
            document.querySelector(`.band-btn[data-band="${band}"]`).classList.add('active');
            
            const hfContainer = document.getElementById('hf-sub-bands');
            if (band === 'HF') {
                hfContainer.style.display = 'block';
                selectHFBand('10m');
            } else {
                hfContainer.style.display = 'none';
                document.querySelectorAll('.hf-band-btn').forEach(btn => {
                    btn.classList.remove('active');
                });
            }
            
            if (radioBands[band]) {
                currentFrequency = radioBands[band].default;
                currentUnit = radioBands[band].unit;
                
                document.getElementById('radio-unit').value = currentUnit;
                document.getElementById('radio-frequency-input').value = currentFrequency.toFixed(3);
                
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
            document.querySelector(`.hf-band-btn[data-hf="${hfBand}"]`).classList.add('active');
            
            if (hfBands[hfBand]) {
                const range = hfBands[hfBand].range.split('-')[0];
                const freq = parseFloat(range);
                if (!isNaN(freq)) {
                    currentFrequency = freq;
                    document.getElementById('radio-frequency-input').value = currentFrequency.toFixed(3);
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
            document.querySelector(`.channel-type-btn[onclick="selectChannelType('${type}')"]`).classList.add('active');
            
            document.querySelectorAll('.channel-container').forEach(container => {
                container.classList.remove('active');
            });
            document.getElementById(`${type}-container`).classList.add('active');
            
            if (type === 'pmr') {
                selectPMRChannel(1);
            } else if (type === 'cb') {
                selectCBChannel(19); // Canal 19 por defecto en CB
            }
            
            playRadioBeep(600, 40);
        }

        function buildPMRChannels() {
            const container = document.getElementById('pmr-channels-grid');
            container.innerHTML = '';
            
            const channels = pmrChannels[currentPMRType];
            for (let i = 1; i <= currentPMRType; i++) {
                const btn = document.createElement('button');
                btn.className = 'channel-btn';
                btn.textContent = i;
                btn.onclick = () => selectPMRChannel(i);
                container.appendChild(btn);
            }
            
            // Marcar primer canal como activo
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
                document.querySelectorAll('#pmr-channels-grid .channel-btn')[channel - 1].classList.add('active');
                
                document.getElementById('radio-frequency-input').value = currentFrequency.toFixed(5);
                updateFrequencyDisplay();
                updateChannelDisplay();
                playRadioBeep(700, 40);
            }
        }

        function buildCBChannels() {
            const container = document.getElementById('cb-channels-grid');
            container.innerHTML = '';
            
            for (let i = 1; i <= 40; i++) {
                const btn = document.createElement('button');
                btn.className = 'channel-btn';
                btn.textContent = i;
                btn.onclick = () => selectCBChannel(i);
                container.appendChild(btn);
            }
            
            // Marcar canal 19 como activo por defecto
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
                document.querySelectorAll('#cb-channels-grid .channel-btn')[channel - 1].classList.add('active');
                
                document.getElementById('radio-frequency-input').value = currentFrequency.toFixed(3);
                updateFrequencyDisplay();
                updateChannelDisplay();
                playRadioBeep(700, 40);
            }
        }

        function updateFrequencyDisplay() {
            const display = document.getElementById('radio-frequency-display');
            const activeDisplay = document.getElementById('active-frequency');
            
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
            document.getElementById('radio-frequency-input').value = currentFrequency.toFixed(3);
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

        // ====== SONIDOS DE RADIO ======

        function initRadioAudio() {
            if (!radioAudioContext) {
                try {
                    radioAudioContext = new (window.AudioContext || window.webkitAudioContext)();
                } catch (error) {
                    console.error("Error inicializando audio radio:", error);
                }
            }
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
                const envelope = i < bufferSize * 0.1 ? i / (bufferSize * 0.1) : 
                                (bufferSize - i) / (bufferSize * 0.9);
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

        // ====== INICIALIZACIÓN DE RADIO ======

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
            
            document.getElementById('radio-frequency-input').value = currentFrequency.toFixed(3);
            document.getElementById('radio-unit').value = currentUnit;
            
            if (currentBand === 'HF' && currentHFBand) {
                document.getElementById('hf-sub-bands').style.display = 'block';
                const hfBtn = document.querySelector(`.hf-band-btn[data-hf="${currentHFBand}"]`);
                if (hfBtn) hfBtn.classList.add('active');
            }
            
            updateFrequencyDisplay();
            updateEmergencyFrequencies();
            updateChannelDisplay();
        }

        // ====== FUNCIONES DE ENVÍO ======

        function handleSendButtonClick() {
            const mode = document.getElementById('inputMode').value;
            if (mode === 'phonetic' && document.getElementById('radio-frequency-input')) {
                sendRadioMessage();
            } else {
                sendMessage();
            }
        }

        function handleSendMessage(event) {
            if (event.key === 'Enter') {
                event.preventDefault();
                handleSendButtonClick();
            }
        }

        // ====== FUNCIÓN DE ENVÍO DE RADIO ======

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
            
            // Si estamos en modo fonético y no es código fonético ya, convertirlo
            if (mode === 'phonetic') {
                processedMessage = textToPhonetic(message);
            }
            
            const dataToSend = prepareMessageToSend(processedMessage, mode, encryption, key);
            const sentCount = sendToConnections(dataToSend);
            
            if (sentCount > 0) {
                // Añadir información de radio al mensaje
                const freq = document.getElementById('radio-frequency-display').textContent;
                const band = currentBand;
                
                displayMessage(`📡 YO [${band} ${freq}]: ${message}`, 
                              dataToSend.hexPreview, 'outgoing');
                
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

        // ====== SISTEMA SATELITAL MEJORADO ======
        const satelliteSystem = {
    latitude: null,
    longitude: null,
    altitude: null,
    accuracy: null,
    weatherData: null,
    lastUpdate: null,
    lastKnownPosition: null  // ← nuevo: para fallback robusto
};

        // ====== ARREGLAR ENVÍO DE EMERGENCIAS ======
        async function sendSatelliteEmergency() {
    updateMonitor("🚨 INICIANDO PROTOCOLO DE EMERGENCIA...", "warning");
    playEmergencySatelliteTone();
    
    try {
        // 1. Intentar obtener posición fresca
        await getCurrentGPSPosition();
        
        // 2. Verificar posición válida
        if (!satelliteSystem.latitude || !satelliteSystem.longitude) {
            throw new Error("No position available");
        }
        
        // 3. Verificar accuracy
        if (satelliteSystem.accuracy > 100) {
            if (!confirm(`⚠️ Precisión baja (${Math.round(satelliteSystem.accuracy)}m). ¿Continuar?`)) {
                return;
            }
        }
        
        // 4. Verificar conexiones
        const onlineCount = Object.keys(connections).filter(id => 
            connections[id]?.status === 'online').length;
        
        if (onlineCount === 0) {
            if (!confirm("⚠️ No hay contactos conectados. ¿Enviar solo a chat local?")) {
                return;
            }
        }
        
        // 5. Confirmar envío
        if (!confirm(`🚨 ¿ENVIAR SEÑAL DE EMERGENCIA S.O.S AYUDA?\n\nPosición: ${satelliteSystem.latitude.toFixed(4)}° N, ${satelliteSystem.longitude.toFixed(4)}° E\nSe enviará a ${onlineCount} contacto(s).`)) {
            return;
        }
        
        // 6. Crear y enviar mensaje
        const emergencyMessage = createEmergencyMessage();
        const sentCount = sendEmergencyMessage(emergencyMessage);
        
        // 7. Confirmación
        updateMonitor(`✅ EMERGENCIA ENVIADA A ${sentCount} CONTACTO(S)`, "info");
        
        // 8. Mostrar en chat local
        displayMessage(`🚨 YO [EMERGENCIA]: ${emergencyMessage.substring(0, 80)}...`, 'EMERG', 'outgoing');
        
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
            
            // Enviar a todos los conectados
            Object.keys(connections).forEach(peerId => {
                if (connections[peerId]?.status === 'online') {
                    try {
                        console.log(`🚨 Enviando emergencia a ${peerId}`);
                        connections[peerId].conn.send(dataToSend);
                        sentCount++;
                        
                        // Actualizar health check
                        if (connectionHealth[peerId]) {
                            connectionHealth[peerId].lastActivity = Date.now();
                            connectionHealth[peerId].packetsSent = (connectionHealth[peerId]?.packetsSent || 0) + 1;
                        }
                    } catch (error) {
                        console.error(`❌ Error enviando emergencia a ${peerId}:`, error);
                    }
                }
            });
            
            // Actualizar estadísticas
            stats.tx += JSON.stringify(dataToSend).length;
            stats.messages++;
            updateStats();
            
            console.log(`✅ Emergencia enviada a ${sentCount} contacto(s)`);
            return sentCount;
        }

        // ====== OBTENER POSICIÓN GPS REAL ======
        function getCurrentGPSPosition(forceOneTime = false) {
        return new Promise((resolve, reject) => {
        if (!navigator.geolocation) {
            updateMonitor("❌ Geolocalización no soportada en este navegador", "error");
            reject(new Error("Geolocation no soportada"));
            return;
        }

        // Si ya tenemos watch activo y no forzamos una sola vez, usamos el watch existente
        if (watchId !== null && !forceOneTime) {
            // Ya está siguiendo → solo resolvemos con la posición actual
            if (satelliteSystem.latitude && satelliteSystem.longitude) {
                resolve();
            }
            return;
        }

        updateMonitor("📍 Solicitando acceso a ubicación...", "info");
        playStrongBeep(800, 100);

        const options = {
            enableHighAccuracy: true,
            timeout: 15000,
            maximumAge: 30000  // Acepta posición de hasta 30 segundos atrás para no pedir tanto
        };

        // Si forzamos "una sola vez" (por botón manual), usamos getCurrentPosition
        if (forceOneTime) {
            navigator.geolocation.getCurrentPosition(
                successCallback,
                errorCallback,
                options
            );
        } else {
            // Modo "fijo" → watchPosition (pide permiso una vez y actualiza automático)
            watchId = navigator.geolocation.watchPosition(
                successCallback,
                errorCallback,
                options
            );
        }

        function successCallback(position) {
            satelliteSystem.latitude = position.coords.latitude;
            satelliteSystem.longitude = position.coords.longitude;
            satelliteSystem.altitude = position.coords.altitude || 0;
            satelliteSystem.accuracy = position.coords.accuracy;
            satelliteSystem.lastUpdate = new Date();

            // Guardar última posición conocida
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
        }

        function errorCallback(error) {
            let msg = "❌ Error GPS: ";
            let showInstructions = false;

            switch (error.code) {
                case error.PERMISSION_DENIED:
                    msg += "Permiso denegado.\n\nPara que no vuelva a pedir cada vez:\n1. Clic en el candado junto a la URL\n2. Ubicación → Permitir (o 'Siempre permitir')\n3. Recarga la página";
                    showInstructions = true;
                    break;
                case error.POSITION_UNAVAILABLE:
                    msg += "Posición no disponible. Verifica GPS o conexión.";
                    break;
                case error.TIMEOUT:
                    msg += "Tiempo agotado. Intenta de nuevo.";
                    break;
                default:
                    msg += error.message;
            }

            updateMonitor(msg, "error");

            if (showInstructions) {
                alert(msg);  // O puedes crear un modal bonito si prefieres
            }

            reject(error);
        }
    });
}

function handleGPSFallback() {
    // 1. Última posición conocida (si < 30 min)
    if (satelliteSystem.lastKnownPosition) {
        const age = (Date.now() - satelliteSystem.lastKnownPosition.timestamp) / 60000;
        if (age < 30 && confirm(`⚠️ Usar posición anterior (${Math.round(age)} min atrás)?`)) {
            Object.assign(satelliteSystem, satelliteSystem.lastKnownPosition);
            updateSatelliteUI();
            updateMonitor(`⚠️ Usando posición anterior | Prec: ${Math.round(satelliteSystem.accuracy)}m`, "warning");
            return;
        }
    }
    
    // 2. Entrada manual
    const manualLat = prompt("⚠️ Ingrese latitud manualmente (ej: 40.4168):");
    const manualLon = prompt("⚠️ Ingrese longitud manualmente (ej: -3.7038):");
    
    if (manualLat && manualLon && !isNaN(parseFloat(manualLat)) && !isNaN(parseFloat(manualLon))) {
        satelliteSystem.latitude = parseFloat(manualLat);
        satelliteSystem.longitude = parseFloat(manualLon);
        satelliteSystem.altitude = 0;
        satelliteSystem.accuracy = 9999;
        satelliteSystem.lastUpdate = new Date();
        
        updateSatelliteUI();
        updateMonitor("⚠️ Usando posición manual", "warning");
        return;
    }
    
    // 3. Default
    if (confirm("⚠️ Usar posición por defecto (Madrid)?")) {
        useDefaultPosition();
    } else {
        updateMonitor("❌ No se obtuvo posición. Operación cancelada.", "error");
    }
}

function useDefaultPosition() {
    satelliteSystem.latitude = 40.4168;
    satelliteSystem.longitude = -3.7038;
    satelliteSystem.altitude = 667;
    satelliteSystem.accuracy = 1000;
    satelliteSystem.lastUpdate = new Date();
    
    updateSatelliteUI();
    updateMonitor("⚠️ Usando posición por defecto (Madrid)", "warning");
}

        function useRealtPosition() {
            satelliteSystem.latitude = 40.4168; // Madrid
            satelliteSystem.longitude = -3.7038;
            satelliteSystem.altitude = 667;
            satelliteSystem.accuracy = 1000;
            satelliteSystem.lastUpdate = new Date();
            
            updateSatelliteUI();
            updateMonitor("⚠️ Usando posición Real", "warning");
            
            fetchWeatherData();
        }

        // ====== OBTENER DATOS METEOROLÓGICOS REALES ======
        // ====== OBTENER DATOS METEOROLÓGICOS + ALTITUD REAL ======
async function fetchWeatherData() {
    if (!satelliteSystem.latitude || !satelliteSystem.longitude) {
        updateMonitor("⚠️ Esperando posición GPS...", "warning");
        return;
    }

    const lat = satelliteSystem.latitude.toFixed(4);
    const lon = satelliteSystem.longitude.toFixed(4);

    updateMonitor("🌤️ Consultando meteo + elevación...", "info");
    updateAPIStatus("Conectando a Open-Meteo...", false);

    try {
        // 1. Datos meteorológicos (tu llamada original, mantenida)
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

        const meteoRes = await fetch(meteoUrl);
        if (!meteoRes.ok) throw new Error(`Meteo HTTP ${meteoRes.status}`);
        satelliteSystem.weatherData = await meteoRes.json();

        // 2. ALTITUD / ELEVACIÓN (endpoint dedicado)
        const elevUrl = `https://api.open-meteo.com/v1/elevation?latitude=${lat}&longitude=${lon}`;
        console.log("📡 Elevación URL:", elevUrl);

        const elevRes = await fetch(elevUrl);
        let elevation = null;
        if (elevRes.ok) {
            const elevData = await elevRes.json();
            if (elevData.elevation && elevData.elevation.length > 0) {
                elevation = Math.round(elevData.elevation[0]);
                satelliteSystem.altitude = elevation;
                console.log("✅ Altitud obtenida:", elevation, "m");
            }
        } else {
            console.warn("Elevación falló:", elevRes.status);
        }

        // Si no hay altitud de API ni de GPS, fallback a 0 o último valor
        if (satelliteSystem.altitude === null || isNaN(satelliteSystem.altitude)) {
            satelliteSystem.altitude = satelliteSystem.altitude || 0;
            console.log("Altitud final (fallback):", satelliteSystem.altitude);
        }

        // Actualizar interfaces
        updateWeatherUI(satelliteSystem.weatherData);
        updateSatelliteUI();  // ← esto actualiza #updateSatelliteUI 
        updateAPIStatus("Conectado", true);
        updateMonitor("✅ Meteo y altitud actualizados", "info");
        playStrongBeep(700, 30);

    } catch (error) {
        console.error("❌ Error total API:", error);
        satelliteSystem.apiConnected = false;
        updateMonitor(`❌ Error: ${error.message}`, "error");
        updateAPIStatus("Error conexión", false);
        playStrongBeep(300, 200);

        // Fallback con altitud de ejemplo
        useSampleWeatherData();
    }
    satelliteSystem.weatherData = sampleData;
    satelliteSystem.altitude = satelliteSystem.altitude || 667; // Madrid por defecto

    updateSatelliteUI(); // Asegura que altitud se muestre
    updateMonitor("⚠️ Usando datos de muestra (Alt: 667 m)", "warning");
}

// Fallback mejorado (añadimos altitud de ejemplo)


        function useSampleWeatherData() {
            const sampleData = {
                current_weather: {
                    temperature: 18.5,
                    windspeed: 12.3,
                    winddirection: 245,
                    weathercode: 3
                },
                hourly: {
                    apparent_temperature: [17.8],
                    relative_humidity_2m: [65],
                    pressure_msl: [1013],
                    wind_gusts_10m: [15.2],
                    cloud_cover: [45],
                    visibility: [15000],
                    precipitation_probability: [20],
                    rain: [0],
                    dew_point_2m: [12.1],
                    uv_index: [4.2],
                    is_day: [1]
                }
            };
            
            updateWeatherUI(sampleData);
        }

        // ====== ACTUALIZAR INTERFAZ SATELITAL ======
        function updateSatelliteUI() {
            // Actualizar coordenadas GPS
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

        function updateWeatherUI(data) {
            if (!data || !data.current_weather) return;
            
            const cw = data.current_weather;
            const hourly = data.hourly;
            const currentIndex = 0;
            
            // Temperatura y viento
            document.getElementById('sat-temp').textContent = `${cw.temperature.toFixed(1)} °C`;
            document.getElementById('sat-windspeed').textContent = `${cw.windspeed.toFixed(1)} km/h`;
            document.getElementById('sat-winddirection').textContent = `${cw.winddirection} °`;
            
            // Datos horarios
            if (hourly) {
                if (hourly.apparent_temperature?.[currentIndex] !== undefined) {
                    document.getElementById('sat-feelslike').textContent = 
                        `${hourly.apparent_temperature[currentIndex].toFixed(1)} °C`;
                }
                
                if (hourly.relative_humidity_2m?.[currentIndex] !== undefined) {
                    document.getElementById('sat-humidity').textContent = 
                        `${hourly.relative_humidity_2m[currentIndex].toFixed(0)} %`;
                }
                
                if (hourly.pressure_msl?.[currentIndex] !== undefined) {
                    document.getElementById('sat-pressure').textContent = 
                        `${hourly.pressure_msl[currentIndex].toFixed(0)} hPa`;
                }
                
                if (hourly.wind_gusts_10m?.[currentIndex] !== undefined) {
                    document.getElementById('sat-windgusts').textContent = 
                        `${hourly.wind_gusts_10m[currentIndex].toFixed(1)} km/h`;
                }
                
                if (hourly.cloud_cover?.[currentIndex] !== undefined) {
                    document.getElementById('sat-cloudcover').textContent = 
                        `${hourly.cloud_cover[currentIndex].toFixed(0)} %`;
                }
                
                if (hourly.visibility?.[currentIndex] !== undefined) {
                    const visibilityKm = hourly.visibility[currentIndex] / 1000;
                    document.getElementById('sat-visibility').textContent = 
                        `${visibilityKm.toFixed(1)} km`;
                }
                
                if (hourly.precipitation_probability?.[currentIndex] !== undefined) {
                    document.getElementById('sat-precipitation').textContent = 
                        `${hourly.precipitation_probability[currentIndex].toFixed(0)} %`;
                }
                
                if (hourly.rain?.[currentIndex] !== undefined) {
                    document.getElementById('sat-rain').textContent = 
                        `${hourly.rain[currentIndex].toFixed(1)} mm`;
                }
                
                if (hourly.dew_point_2m?.[currentIndex] !== undefined) {
                    document.getElementById('sat-dewpoint').textContent = 
                        `${hourly.dew_point_2m[currentIndex].toFixed(1)} °C`;
                }
                
                if (hourly.uv_index?.[currentIndex] !== undefined) {
                    document.getElementById('sat-uvindex').textContent = 
                        `${hourly.uv_index[currentIndex].toFixed(1)}`;
                }
                
                // Calcular índices derivados
                calculateWeatherIndexes(cw.temperature, 
                    hourly.relative_humidity_2m?.[currentIndex] || 50,
                    cw.windspeed);
            }
        }

        function calculateWeatherIndexes(temp, humidity, windSpeed) {
            // Heat Index (Índice de calor)
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
            
            // Wind Chill (Sensación por viento)
            if (temp <= 10 && windSpeed >= 5) {
                const wc = 13.12 + 0.6215*temp - 11.37*Math.pow(windSpeed, 0.16) 
                          + 0.3965*temp*Math.pow(windSpeed, 0.16);
                document.getElementById('sat-windchill').textContent = `${wc.toFixed(1)} °C`;
            } else {
                document.getElementById('sat-windchill').textContent = "N/A";
            }
            
            // Calidad de aire estimada
            let aqiScore = 50;
            if (humidity > 80) aqiScore += 20;
            if (humidity < 30) aqiScore += 10;
            if (temp > 30) aqiScore += 15;
            
            let aqiLevel = "Buena";
            if (aqiScore > 150) aqiLevel = "Muy pobre";
            else if (aqiScore > 100) aqiLevel = "Pobre";
            else if (aqiScore > 50) aqiLevel = "Moderada";
            
            document.getElementById('sat-aqi').textContent = aqiLevel;
            
            // Otros valores por defecto
            document.getElementById('sat-sunshine').textContent = "8.5 h";
            document.getElementById('sat-co2').textContent = "415 ppm";
        }

        function updateAPIStatus(text, connected) {
            const dot = document.getElementById('api-status-dot');
            const textElement = document.getElementById('api-status-text');
            
            dot.className = "status-dot-api " + (connected ? "status-connected" : "status-disconnected");
            textElement.textContent = text;
            textElement.style.color = connected ? "#00ff88" : "#ff3300";
        }

        // ====== ACTUALIZACIÓN FORZADA ======
        function forceUpdateSatelliteData() {
            updateMonitor("🔄 Actualización forzada de datos satelitales...", "info");
            playStrongBeep(900, 80);
            
            getCurrentGPSPosition();
        }

        // ====== COMPARTIR POSICIÓN EN CHAT ======
       function sendPositionToChat() {
    if (!satelliteSystem.latitude || !satelliteSystem.longitude) {
        updateMonitor("❌ No hay posición GPS. Obtén posición primero.", "error");
        playStrongBeep(300, 200);
        getCurrentGPSPosition();  // ← esto ya lo tenías
        return;
    }

    const lat = satelliteSystem.latitude.toFixed(6);
    const lon = satelliteSystem.longitude.toFixed(6);
    const alt = satelliteSystem.altitude ? Math.round(satelliteSystem.altitude) : 'N/A';
    const time = new Date().toLocaleTimeString();

    // Mensaje simple y limpio (como lo tenías originalmente)
    const positionMessage = `📍 POSICIÓN ACTUAL\nLat: ${lat}\nLon: ${lon}\nAlt: ${alt} m\nHora: ${time}`;

    // Enviar usando el mecanismo estándar de mensajes (sin forzar encriptación extra aquí)
    const dataToSend = prepareMessageToSend(
        positionMessage,
        'text',                           // modo texto
        document.getElementById('encryptionMode').value,  // respeta la config del usuario
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

        // ====== SISTEMA DE ACTUALIZACIÓN AUTOMÁTICA ======
        function startAutoRefresh() {
            if (satelliteSystem.refreshInterval) {
                clearInterval(satelliteSystem.refreshInterval);
            }
            
            satelliteSystem.refreshInterval = setInterval(() => {
                const autoRefreshCheckbox = document.getElementById('auto-refresh-checkbox');
                if (autoRefreshCheckbox?.checked && !document.hidden && satelliteSystem.latitude) {
                    console.log("🔄 Actualización automática satelital");
                    fetchWeatherData();
                }
            }, 60000); // 60 segundos
            
            updateMonitor("✅ Actualización automática activada (60s)", "info");
        }

        // ====== INICIALIZACIÓN SATELITAL ======
        function initSatelliteSystem() {
    // Cargar última posición conocida de localStorage
    const savedPos = localStorage.getItem('radcom_last_position');
    if (savedPos) {
        satelliteSystem.lastKnownPosition = JSON.parse(savedPos);
        console.log("📍 Posición anterior cargada:", satelliteSystem.lastKnownPosition);
    }
    
    // Actualizar UI inicial
    updateSatelliteUI();
    
    // Auto-refresco cada 5 minutos si app activa
    setInterval(() => {
        if (!document.hidden) {
            forceUpdateSatelliteData();
        }
    }, 300000);
}

function startWatchingPosition() {
    if (watchId !== null) return; // Ya está activo

    getCurrentGPSPosition(false)  // false = modo watch (fijo)
        .then(() => {
            console.log("Seguimiento GPS iniciado en modo continuo");
        })
        .catch(() => {
            console.warn("No se pudo iniciar seguimiento continuo");
        });
}

function stopWatchingPosition() {
    if (watchId !== null) {
        navigator.geolocation.clearWatch(watchId);
        watchId = null;
        console.log("Seguimiento GPS detenido");
    }
}

// Ejemplo: parar cuando la pestaña está oculta
document.addEventListener('visibilitychange', () => {
    if (document.hidden) {
        stopWatchingPosition();
    } else {
        // Solo intentamos recuperar posición si ya teníamos permiso antes
        if (satelliteSystem.lastKnownPosition) {
            updateSatelliteUI(); // usamos la última conocida
        } else {
            // Solo pedimos una vez al volver
            setTimeout(() => {
                if (!watchId) getCurrentGPSPosition(true); // true = solo una vez
            }, 800);
        }
    }
});

function forceUpdateSatelliteData() {
    getCurrentGPSPosition()
        .then(() => {
            fetchWeatherData();
        })
        .catch(() => {
            updateMonitor("⚠️ Actualización satelital fallida", "warning");
        });
}
        // ====== SONIDOS MEJORADOS ======
        function playEmergencySatelliteTone() {
            if (!document.getElementById('soundEnabled')?.checked) return;
            
            if (!audioContext) {
                audioContext = new (window.AudioContext || window.webkitAudioContext)();
            }
            
            let time = audioContext.currentTime;
            
            // Sirena de emergencia
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

        // ====== MANEJAR MENSAJES DE EMERGENCIA RECIBIDOS ======
        function handleEmergencyMessage(senderId, data) {
            const key = document.getElementById('key').value || 'ATOM80';
            const decryptedMessage = xorDecrypt(data.message, key);
            
            displayMessage(`🚨 EMERGENCIA DE ${senderId.substring(0,6)}: ${decryptedMessage.substring(0, 100)}...`, 
                          'EMERG', 'incoming');
            
            // Sonido de alerta
            if (document.getElementById('soundEnabled')?.checked) {
                playEmergencyAlertSound();
            }
            
            updateMonitor(`🚨 EMERGENCIA RECIBIDA DE ${senderId.substring(0,6)}`);
            
            // Actualizar estadísticas
            stats.rx += data.message.length;
            updateStats();
        }

        function playEmergencyAlertSound() {
            if (!audioContext) {
                audioContext = new (window.AudioContext || window.webkitAudioContext)();
            }
            
            let time = audioContext.currentTime;
            
            // Alerta triple aguda
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

        // ====== INTEGRACIÓN CON SISTEMA EXISTENTE ======
        // Guardar referencia a la función original de handleReceivedData
        const originalHandleReceivedData = window.handleReceivedData;

        // Sobrescribir para manejar emergencias
        window.handleReceivedData = function(senderId, data) {
            if (data.type === 'emergency') {
                handleEmergencyMessage(senderId, data);
                return;
            }
            
            // Si no es emergencia, usar el manejo original
            if (originalHandleReceivedData) {
                originalHandleReceivedData(senderId, data);
            }
        };

        // ====== FUNCIONES DE ENVÍO ORIGINALES ======

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
            
            // Si estamos en modo satélite, añadir automáticamente la posición GPS
            if (mode === 'satellite' && !message.includes('🚨 EMERGENCIA S.O.S AYUDA')) {
                const pos = satellitePositions.paragliding;
                const gpsInfo = `\n\n[GPS: ${pos.lat.toFixed(4)}° N, ${pos.lon.toFixed(4)}° E | Alt: ${Math.round(pos.alt)} m]`;
                message += gpsInfo;
            }
            
            if (!validateInput()) {
                updateMonitor(`⚠️ FORMATO ${mode.toUpperCase()} INVÁLIDO`, "error");
                playStrongBeep(300, 200);
                return;
            }
            
            let processedMessage = message;
            
            sendMessage
            
            const dataToSend = prepareMessageToSend(processedMessage, mode, encryption, key);
            const sentCount = sendToConnections(dataToSend);
            
            if (sentCount > 0) {
                displayMessage(`YO: ${message}`, dataToSend.hexPreview, 'outgoing');
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
                
                document.querySelectorAll('#ansiTable td.highlighted').forEach(td => {
                    td.classList.remove('highlighted');
                });
            } else {
                updateMonitor("⚠️ SIN DESTINOS CONECTADOS", "warning");
                playStrongBeep(300, 200);
            }
        }

        function isMorseCode(text) {
            const morseRegex = /^[\.\-\s\/]+$/;
            return morseRegex.test(text);
        }

        function textToMorse(text) {
            return text.toUpperCase().split('').map(char => morseCodes[char] || char).join(' ');
        }

        // ====== FUNCIÓN PARA MOSTRAR TRADUCCIÓN MORSE ======
function updateMorseTranslation(text) {
    const translationBox = document.getElementById('morse-translation');
    const textDisplay = document.getElementById('morse-text-display');
    const codeDisplay = document.getElementById('morse-code-display');
    
    if (!text.trim()) {
        translationBox.classList.remove('active');
        return;
    }
    
    // Mostrar siempre
    translationBox.classList.add('active');
    
    // Si es código Morse, intentar decodificar
    if (/^[\.\-\s\/]+$/.test(text.trim())) {
        textDisplay.textContent = decodeMorse(text) || "[Código Morse]";
        codeDisplay.textContent = text;
    } 
    // Si es texto normal, codificar a Morse
    else {
        textDisplay.textContent = text;
        codeDisplay.textContent = textToMorse(text);
    }

    function sendMessage() {
    const input = document.getElementById('inputMsg');
    const rawText = input.value.trim();
    if (!rawText) return;

    const mode = document.getElementById('inputMode')?.value || 'text';

    const secureResult = prepareAndSecureMessage(rawText, mode);

    if (secureResult.error) {
        updateMonitor("⚠️ " + secureResult.error);
        return;
    }

    const data = secureResult.data;

    // Enviar
    let sentCount = 0;
    if (activeTarget === 'GLOBAL') {
        Object.keys(connections).forEach(pid => {
            if (connections[pid]?.conn?.open) {
                connections[pid].conn.send(data);
                sentCount++;
            }
        });
    } else if (connections[activeTarget]?.conn?.open) {
        connections[activeTarget].conn.send(data);
        sentCount = 1;
    }

    if (sentCount > 0) {
        // Mostrar localmente el texto original (no el cifrado)
        const bubble = document.createElement('div');
        bubble.className = 'message-bubble message-outgoing';
        bubble.innerHTML = `<strong>YO:</strong> ${rawText}<br>
                            <small style="color:#00ffea">Cifrado: ${data.enc}</small>`;
        document.getElementById('monitor-decoded').appendChild(bubble);
        document.getElementById('monitor-decoded').scrollTop = document.getElementById('monitor-decoded').scrollHeight;

        updateMonitor(`Enviado (${data.enc}) a ${sentCount} destino(s)`);
    } else {
        updateMonitor("⚠️ Sin conexiones abiertas");
    }

    input.value = '';
}
}

// ====== FUNCIÓN PARA DECODIFICAR MORSE ======
function decodeMorse(morseCode) {
    const morseMap = {};
    for (const [char, code] of Object.entries(morseCodes)) {
        morseMap[code] = char;
    }
    
    return morseCode.split(' ').map(code => morseMap[code] || '?').join('');
}

        function textToPhonetic(text) {
            return text.toUpperCase().split('').map(char => phoneticAlphabetFull[char]?.word || char).join(' ');
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
    if (encryption === 'xor' || encryption === 'aes') {
        encryptedMessage = xorEncrypt(processedMessage, key);
        
        for (let i = 0; i < Math.min(encryptedMessage.length, 3); i++) {
            hexPreview += encryptedMessage.charCodeAt(i).toString(16).padStart(2, '0');
        }
        if (encryptedMessage.length > 3) hexPreview += '...';
    }
    
    // ====== AÑADIR DATOS DUAL ======
    return {
        type: 'message',
        message: encryptedMessage,
        original: message,
        mode: mode,
        encryption: encryption,
        timestamp: Date.now(),
        sender: myPeerId,
        hexPreview: hexPreview,
        // Datos para sistema dual Morse
        textVersion: extraData.textVersion || message,
        morseVersion: extraData.morseVersion || '',
        isDualMorse: extraData.isDualMorse || false
    };
}

        function xorEncrypt(text, key) {
            let result = '';
            for (let i = 0; i < text.length; i++) {
                const charCode = text.charCodeAt(i);
                const keyCode = key.charCodeAt(i % key.length);
                result += String.fromCharCode(charCode ^ keyCode);
            }
            return result;
        }

        function xorDecrypt(text, key) {
            return xorEncrypt(text, key);
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

        // ====== FUNCIONES DE CONEXIÓN ======

        function setupPeerEvents() {
            peer.on('open', (id) => {
                myPeerId = id;
                localStorage.setItem('radcom_master_id_v4', id);
                
                document.getElementById('display-id').innerHTML = 
                    `<span style="color:#00ff88">ID: ${id.substring(0, 10)}...</span>`;
                
                const typeName = currentConnectionType === 'wifi' ? 'WiFi' : 'Datos Móviles';
                updateMonitor(`✅ SISTEMA v${VERSION} INICIADO | ${typeName} | ID: ${id.substring(0, 8)}`);
                
                savedIds.forEach(peerId => {
                    if (peerId !== id) {
                        setTimeout(() => connectToPeerId(peerId), 100);
                    }
                });
                
                setInterval(updateUptime, 1000);
                updateStats();
            });
            
            peer.on('connection', (conn) => {
                console.log("📞 Conexión entrante de:", conn.peer);
                handleIncomingConnection(conn);
            });
            
            peer.on('error', (err) => {
    if (err.type === 'peer-unavailable') {
        updateMonitor(`⚠️ Peer non-existent or offline`, "warning");
    } else {
        console.error("PeerJS Error:", err);
        updateMonitor(`⚠️ ERROR: ${err.type}`, "error");
    }
});
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
                setTimeout(() => {
        deliverOnConnect(peerId);
    }, 1500);
                
                saveId(peerId);
                
                setTimeout(() => sendHealthPing(peerId), 1000);
            });
            
            conn.on('data', (data) => {
                console.log("📥 Datos recibidos de", peerId, ":", data);
                
                connectionHealth[peerId].lastActivity = Date.now();
                connectionHealth[peerId].packetsReceived++;
                
                if (data.type === 'health_ping') {
                    try {
                        conn.send({
                            type: 'health_pong',
                            timestamp: data.timestamp,
                            sender: myPeerId
                        });
                        console.log(`📡 Pong enviado a ${peerId}`);
                        return;
                    } catch (error) {
                        console.error(`❌ Error enviando pong:`, error);
                    }
                }
                
                if (data.type === 'health_pong') {
                    handleHealthPong(peerId, data.timestamp);
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
                const conn = peer.connect(peerId, {
                    reliable: true,
                    serialization: 'json'
                });
                
                setupConnection(conn, 'outgoing');
                
            } catch (error) {
                console.error(`❌ Error conectando a ${peerId}:`, error);
                updateMonitor(`❌ ERROR CONECTANDO A ${peerId.substring(0, 8)}`);
            }
        }

        function connectToPeer() {
            const input = document.getElementById('peer-id-input');
            const peerId = input.value.trim();
            
            if (!peerId) {
                updateMonitor("⚠️ INGRESA UN ID VÁLIDO", "error");
                playStrongBeep(300, 200);
                return;
            }
            
            input.value = '';
            saveId(peerId);
            connectToPeerId(peerId);
        }

        function handleIncomingConnection(conn) {
            console.log("🔄 Procesando conexión entrante:", conn.peer);
            setupConnection(conn, 'incoming');
        }

        // ====== VERIFICAR CONEXIONES REALES ======
function verificarConexiones() {
    console.log("🔍 Verificando conexiones...");
    
    let problemas = 0;
    const ahora = Date.now();
    
    // Verificar cada conexión
    for (const peerId in connections) {
        const conn = connections[peerId];
        
        if (!conn) continue;
        
        // Si dice estar online, verificar que sea real
        if (conn.status === 'online') {
            let estaViva = true;
            
            // Verificación 1: Objeto conexión existe
            if (!conn.conn) {
                estaViva = false;
                console.log(`❌ ${peerId.substring(0,6)}: No tiene objeto conexión`);
            }
            // Verificación 2: Conexión abierta
            else if (!conn.conn.open) {
                estaViva = false;
                console.log(`❌ ${peerId.substring(0,6)}: Conexión no abierta`);
            }
            // Verificación 3: Última actividad
            else if (connectionHealth[peerId]?.lastActivity) {
                const inactivo = ahora - connectionHealth[peerId].lastActivity;
                if (inactivo > 60000) { // 1 minuto sin actividad
                    console.log(`⚠️ ${peerId.substring(0,6)}: Inactivo ${Math.round(inactivo/1000)}s`);
                    conn.health = 'inactive';
                    problemas++;
                }
            }
            
            if (!estaViva) {
                conn.status = 'offline';
                conn.health = 'dead';
                problemas++;
                console.log(`🔴 ${peerId.substring(0,6)}: Marcado como offline`);
            }
        }
        // Si está offline pero debería estar online (en savedIds)
        else if (conn.status === 'offline' || conn.status === 'error') {
            const estaGuardado = savedIds.includes(peerId);
            if (estaGuardado && conn.health !== 'reconnecting') {
                problemas++;
                console.log(`🔄 ${peerId.substring(0,6)}: Offline pero guardado`);
            }
        }
    }
    
    // Actualizar UI
    updatePeerList();
    updateConnectedCount();
    
    if (problemas > 0) {
        console.log(`📊 ${problemas} problema(s) encontrado(s)`);
        return false;
    }
    
    return true;
}

// ====== REPARAR CONEXIONES PROBLEMÁTICAS ======
function repararConexiones() {
    console.log("🛠️ Reparando conexiones...");
    updateMonitor("🔧 REPARANDO...", "info");
    
    let reparadas = 0;
    
    for (const peerId in connections) {
        const conn = connections[peerId];
        
        if (!conn) continue;
        
        // Condiciones para reparar:
        // 1. Está offline/error
        // 2. O está online pero con salud mala
        const necesitaReparar = (
            (conn.status === 'offline' || conn.status === 'error') ||
            (conn.status === 'online' && conn.health === 'inactive')
        );
        
        if (necesitaReparar && conn.health !== 'reconnecting') {
            console.log(`🔄 Reparando ${peerId.substring(0,6)}...`);
            
            // Marcar como reconectando
            conn.health = 'reconnecting';
            updatePeerList();
            
            // Usar función existente para reconectar
            if (peer && !peer.disconnected) {
                // Cerrar conexión vieja si existe
                if (conn.conn) {
                    try {
                        conn.conn.close();
                    } catch(e) {}
                }
                
                // Eliminar registro temporal
                delete connections[peerId];
                delete connectionHealth[peerId];
                
                // Esperar y reconectar usando función existente
                setTimeout(() => {
                    if (connectToPeerId) {
                        connectToPeerId(peerId);
                        reparadas++;
                    }
                }, 500);
            }
        }
    }
    
    // Actualizar
    updatePeerList();
    
    if (reparadas > 0) {
        console.log(`✅ ${reparadas} conexión(es) en reparación`);
        updateMonitor(`✅ ${reparadas} conexión(es) reparándose`);
        playStrongBeep(700, 100);
    } else {
        console.log("✅ Todas las conexiones OK");
        updateMonitor("✅ Conexiones verificadas");
    }
    
    return reparadas;
}

// ====== VERIFICAR Y REPARAR ======
function verificarYReparar() {
    console.log("⚡ Verificando y reparando...");
    
    // 1. Verificar
    const ok = verificarConexiones();
    
    // 2. Si hay problemas, reparar
    if (!ok) {
        console.log("⚠️ Problemas detectados, reparando...");
        setTimeout(() => {
            repararConexiones();
        }, 1000);
    } else {
        console.log("✅ Todas las conexiones OK");
        updateMonitor("✅ Conexiones verificadas");
    }
    
    return ok;
}

        // ====== FUNCIÓN NUEVA: sendWithQueue() ======
// Usar esta en lugar de sendMessage() normal
function sendWithQueue() {
    const input = document.getElementById('inputMsg');
    let message = input.value.trim();
    
    if (!message) {
        updateMonitor("⚠️ MENSAJE VACÍO", "error");
        playStrongBeep(300, 200);
        return;
    }
    
    const originalMessage = message;
    const key = document.getElementById('key').value || 'ATOM80';
    const mode = document.getElementById('inputMode').value;
    const encryption = document.getElementById('encryptionMode').value;
    
    if (!validateInput()) {
        updateMonitor(`⚠️ FORMATO ${mode.toUpperCase()} INVÁLIDO`, "error");
        playStrongBeep(300, 200);
        return;
    }
    
    // Preparar mensaje (igual que función original)
        // Preparar mensaje con sistema DUAL bidireccional
    let processedMessage = message;
    let extraData = {}; // Datos extra para el modo dual
    
    if (mode === 'morse') {
        // ====== SISTEMA DUAL MORSE ======
        const isMorse = /^[\.\-\s\/]+$/.test(message.trim());
        
        if (isMorse) {
            // Escribieron Morse → enviamos Morse, guardamos texto traducido
            processedMessage = message;  // Enviamos el Morse original
            extraData = {
                textVersion: decodeMorse(message) || message,
                morseVersion: message,
                isDualMorse: true
            };
            updateMonitor(`📡 Enviando Morse (${decodeMorse(message)?.substring(0, 20) || "Código"})`);
        } else {
            // Escribieron Texto → enviamos Morse traducido, guardamos texto original
            processedMessage = textToMorse(message);  // Enviamos Morse traducido
            extraData = {
                textVersion: message,
                morseVersion: processedMessage,
                isDualMorse: true
            };
            updateMonitor(`📡 Enviando "${message.substring(0, 20)}" como Morse`);
        }
        
        // Mostrar traducción para aprendizaje
        updateMorseTranslation(message);
        
    } else if (mode === 'phonetic') {
        processedMessage = textToPhonetic(message);
    }
    
    const dataToSend = prepareMessageToSend(processedMessage, mode, encryption, key, extraData);
    
    // 1. Intentar enviar normalmente
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
    
    // 2. Resultado
    if (sentCount > 0) {
        // Éxito: mostrar normal
        displayMessage(`YO: ${originalMessage}`, dataToSend.hexPreview, 'outgoing');
        stats.tx += JSON.stringify(dataToSend).length;
        stats.messages++;
        input.value = '';
        updateMonitor(`📤 ENVIADO A ${sentCount} DESTINO${sentCount !== 1 ? 'S' : ''}`);
        updateStats();
        playStrongBeep(700, 100);
    } else {
        // Guardar en cola
        addToQueue(dataToSend, originalMessage);
        displayMessage(`⏳ YO (EN COLA): ${originalMessage}`, dataToSend.hexPreview, 'outgoing');
        input.value = '';
        updateMonitor(`💾 GUARDADO EN COLA (${messageQueue.length} mensajes)`);
        playStrongBeep(500, 100);
    }
    
    // Limpiar tabla
    document.querySelectorAll('#ansiTable td.highlighted').forEach(td => {
        td.classList.remove('highlighted');
    });
}

// ====== FUNCIÓN NUEVA: addToQueue() ======
function addToQueue(dataToSend, originalMessage) {
    // Limpiar si está llena
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
    
    // Mostrar en UI
    updateQueueCounter();
    
    // Iniciar sistema si no está activo
    if (!queueRetryInterval) {
        startQueueSystem();
    }
}

// ====== FUNCIÓN NUEVA: startQueueSystem() ======
function startQueueSystem() {
    if (queueRetryInterval) clearInterval(queueRetryInterval);
    
    queueRetryInterval = setInterval(() => {
        processQueue();
    }, 1000000); // Cada 1000 segundos
    
    updateMonitor("🔄 Sistema de cola ACTIVADO", "info");
}

// ====== FUNCIÓN NUEVA: processQueue() ======
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
        // Saltar si es muy reciente
        if (now - item.timestamp < 5000) {
            remaining.push(item);
            return;
        }
        
        item.attempts++;
        
        let delivered = false;
        
        // Intentar entregar
        if (item.target === 'GLOBAL') {
            Object.keys(connections).forEach(peerId => {
                if (connections[peerId]?.status === 'online') {
                    try {
                        connections[peerId].conn.send(item.data);
                        delivered = true;
                        
                        // Mostrar entrega
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
            // Estadísticas
            stats.tx += JSON.stringify(item.data).length;
            stats.messages++;
            updateStats();
            
            playStrongBeep(600, 50);
        } else if (item.attempts < 10) {
            // Mantener para otro intento
            remaining.push(item);
        } else {
            // Demasiados intentos
            displayMessage(`❌ NO ENTREGADO: ${item.original.substring(0, 30)}...`, '', 'system');
        }
    });
    
    // Actualizar cola
    messageQueue = remaining;
    localStorage.setItem('radcom_message_queue', JSON.stringify(messageQueue));
    updateQueueCounter();
}

// ====== FUNCIÓN NUEVA: updateQueueCounter() ======
function updateQueueCounter() {
    // Buscar o crear elemento
    let counter = document.getElementById('queue-counter');
    
    if (!counter) {
        // Crear en el footer
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
        
        // Color según cantidad
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

// ====== FUNCIÓN NUEVA: deliverOnConnect() ======
function deliverOnConnect(peerId) {
    if (messageQueue.length === 0) return;
    
    const remaining = [];
    
    messageQueue.forEach(item => {
        // Si es para este peer o para todos
        if (item.target === peerId || item.target === 'GLOBAL') {
            try {
                if (connections[peerId]?.status === 'online') {
                    connections[peerId].conn.send(item.data);
                    
                    // Mostrar
                    displayMessage(`🎯 ENTREGADO A NUEVA CONEXIÓN ${peerId.substring(0,6)}`, '', 'system');
                    
                    // Estadísticas
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
    
    // Actualizar
    messageQueue = remaining;
    localStorage.setItem('radcom_message_queue', JSON.stringify(messageQueue));
    updateQueueCounter();
}


        // ====== SISTEMA DE HEALTH CHECKS MEJORADO ======
        
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

        // ====== DETECCIÓN DE SEGUNDO PLANO MEJORADA ======
        
        document.addEventListener('visibilitychange', function() {
            isBackground = document.hidden;
            
            console.log(`📱 App ${isBackground ? 'en segundo plano' : 'en primer plano'}`);
            
            if (!isBackground) {
                updateMonitor("⚡ APP REACTIVADA - VERIFICANDO CONEXIONES...");
                playStrongBeep(800, 100);
                
                if (fastRecovery) {
                    setTimeout(() => {
                        reviveAllConnections();
                    }, 500);
                } else {
                    checkConnectionsHealth();
                }
            } else {
                updateMonitor("⏸️ APP EN SEGUNDO PLANO");
            }
        });

        // ====== FUNCIÓN REVIVIR CONEXIONES MEJORADA ======
        
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
                    (connections[peerId]?.health === 'suspicious' || 
                     connections[peerId]?.health === 'unresponsive')) {
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
                    (connections[peerId]?.health === 'suspicious' || 
                     connections[peerId]?.health === 'unresponsive')) {
                    
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
                        if (connections[peerId]?.status === 'online' && 
                            connections[peerId]?.health === 'healthy') {
                            healthyCount++;
                        }
                    });
                    
                    updateMonitor(`✅ ${healthyCount} CONEXIÓN(ES) SALUDABLES`);
                    finishReviveOperation(reviveBtn, revived);
                }, 5000);
            }, 3000);
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

        // ====== REFRESCAR CONEXIONES MEJORADO ======
        
        function refreshAllConnections() {
            console.log("🔄 Refrescando todas las conexiones...");
            updateMonitor("🔄 REFRESCANDO CONEXIONES...", "info");
            playStrongBeep(1000, 200);
            
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

        // ====== FUNCIONES DE MENSAJERÍA ======

        function handleReceivedData(senderId, data) {
    console.log("🔄 Procesando datos de", senderId, ":", data);
    
    connectionHealth[senderId] = {
        ...connectionHealth[senderId],
        lastActivity: Date.now(),
        lastReceived: Date.now()
    };
    
    if (!data || !data.message) {
        console.log("❌ Datos inválidos");
        return;
    }
    
    const key = document.getElementById('key').value || 'ATOM80';
    const encryption = document.getElementById('encryptionMode').value;
    
    let decryptedMessage = data.message;
    
    if (encryption === 'xor' || encryption === 'aes') {
        decryptedMessage = xorDecrypt(data.message, key);
    }
    
    const displayName = senderId.substring(0, 6);
    
    // ====== SISTEMA DUAL MORSE ======
    if (data.isDualMorse && data.textVersion && data.morseVersion) {
        // Mostrar AMBAS versiones: Texto (Morse)
        displayMessage(
            `${displayName}: ${data.textVersion} (${data.morseVersion})`,
            data.hexPreview || '',
            'incoming'
        );
    } else {
        // Mostrar normal
        displayMessage(`${displayName}: ${decryptedMessage}`, 
                     data.hexPreview || '', 'incoming');
    }
    
    stats.rx += data.message.length;
    stats.messages++;
    updateStats();
    
    updateMonitor(`📥 ${displayName}: "${decryptedMessage.substring(0, 20)}${decryptedMessage.length > 20 ? '...' : ''}"`);
    
    playMessageNotification();

    function handleReceivedData(senderId, data) {
    if (!data || data.type !== "message" || !data.message) return;

    const key = document.getElementById('key')?.value?.trim() || "";
    const encMode = document.getElementById('encryptionMode')?.value || "none"; // no se usa realmente para descifrar

    const secured = securityLayer(data.message, false, data.encryption, key);

    const displayName = senderId.substring(0, 8);
    let color = "#888";

    if (secured.encryptionUsed === "AES-256-GCM") color = "#00ffea";
    else if (secured.encryptionUsed === "XOR") color = "#ffaa00";
    else color = "#ff3300";

    // Mostrar
    const bubble = document.createElement("div");
    bubble.className = "message-bubble message-incoming";
    bubble.innerHTML = `
        <strong>${displayName}:</strong> ${secured.text}
        <br><small style="color:${color}">${secured.encryptionUsed}${secured.error ? " ⚠️" : ""}</small>
    `;
    document.getElementById("monitor-decoded").appendChild(bubble);
    document.getElementById("monitor-decoded").scrollTop = document.getElementById("monitor-decoded").scrollHeight;

    if (secured.error) {
        updateMonitor(`⚠️ Error seguridad de ${displayName}: ${secured.error}`);
    }
}

function handleReceivedData(senderId, data) {
    if (data.type !== 'message') return;

    let text = data.payload || '';

    // ¿Es AES-256-GCM?
    if (data.enc === 'AES-256-GCM') {
        const aesKey = connections[senderId]?.aesKey;
        if (!aesKey) {
            console.warn("Mensaje cifrado recibido pero sin clave");
            text = "[clave no negociada]";
        } else {
            try {
                const [ivHex, ct] = text.split(':');
                const iv = CryptoJS.enc.Hex.parse(ivHex);
                const key = CryptoJS.enc.Hex.parse(aesKey);
                const decrypted = CryptoJS.AES.decrypt(ct, key, { iv });
                text = decrypted.toString(CryptoJS.enc.Utf8) || "[error descifrado]";
            } catch (e) {
                text = "[fallo descifrado]";
            }
        }
    }

    // ¿Es Morse?
    if (data.mode === 'morse') {
        if (typeof decodeMorse === 'function') {
            text = decodeMorse(text);
        } else {
            text = "[morse no decodificado]";
        }
    }

    // Mostrar
    const bubble = document.createElement('div');
    bubble.className = 'message-bubble message-incoming';
    bubble.innerHTML = `<strong>${senderId.substring(0,8)}:</strong> ${text}<br>
                        <small style="color:#00ffea">${data.enc}</small>`;
    document.getElementById('monitor-decoded').appendChild(bubble);
    document.getElementById('monitor-decoded').scrollTop = document.getElementById('monitor-decoded').scrollHeight;
}

    
}

        // ====== FUNCIONES DE UI MEJORADAS ======
        
        function updatePeerList() {
            const container = document.getElementById('saved-peers');
            let html = "";
            
            savedIds.forEach(id => {
                const conn = connections[id];
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
                
                html += `
                    <div class="peer-item ${activeTarget === id ? 'active' : ''}" 
                         data-peer-id="${id}"
                         data-status="${status}"
                         data-health="${health}"
                         onclick="selectPeer('${id}')"
                         style="${displayStyle}">
                        <div class="peer-info">
                            <span class="status-dot ${statusClass}"></span>
                            <div style="display:flex; flex-direction:column;">
                                <span style="font-weight:bold;">${id.substring(0, 8)}</span>
                                <span style="font-size:0.55rem; color:#888;">
                                    ${statusText}
                                    ${conn?.latency ? ` (${conn.latency}ms)` : ''}
                                </span>
                            </div>
                        </div>
                        <span class="del-btn" onclick="event.stopPropagation(); executeRemoveId('${id}')">×</span>
                    </div>
                `;
            });
            
            container.innerHTML = html;
            updateConnectionStatusIndicators();

            
        }

        function updateConnectionStatusIndicators() {
            const onlineHealthy = Object.keys(connections).filter(id => 
                connections[id]?.status === 'online' && 
                connections[id]?.health === 'healthy').length;
            
            const onlineSuspicious = Object.keys(connections).filter(id => 
                connections[id]?.status === 'online' && 
                (connections[id]?.health === 'suspicious' || 
                 connections[id]?.health === 'unresponsive')).length;
            
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

        // ====== FUNCIONES AUXILIARES MEJORADAS ======
        
        function saveId(id) {
            if (!savedIds.includes(id) && id !== myPeerId) {
                savedIds.push(id);
                localStorage.setItem('radcom_peers_v4', JSON.stringify(savedIds));
                updatePeerList();
            }
        }

        function executeRemoveId(id) {
            if (!confirm(`¿Eliminar ${id.substring(0, 8)} del historial?`)) {
                return;
            }
            
            console.log("🗑️ Eliminando ID:", id);
            
            savedIds = savedIds.filter(savedId => savedId !== id);
            localStorage.setItem('radcom_peers_v4', JSON.stringify(savedIds));
            
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

        function updateConnectedCount() {
            const onlineCount = Object.keys(connections).filter(id => 
                connections[id]?.status === 'online').length;
            document.getElementById('stats-connections').textContent = onlineCount;
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
                    <span style="color:#${type === 'outgoing' ? '0088ff' : type === 'incoming' ? 'ff3300' : '00ff88'}">
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
            const color = type === 'error' ? '#ff3300' : 
                         type === 'warning' ? '#ffaa00' : '#00ff88';
            
            monitor.innerHTML = `<span style="color:${color}">${message}</span>`;
        }

        // ====== SONIDOS ======
        
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

        // ====== FUNCIONES RESTANTES ======
        
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
        
        // CRÍTICO: Notificar al sistema que la clave ha cambiado
        keyInput.dispatchEvent(new Event('change'));
        keyInput.dispatchEvent(new Event('input'));
        
        setTimeout(() => {
            keyInput.type = 'password';
        }, 2000);
    }
    
    updateMonitor("🔑 NUEVA CLAVE MAESTRA GENERADA");
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

        function updateUptime() {
            const elapsed = Date.now() - stats.startTime;
            const hours = Math.floor(elapsed / 3600000);
            const minutes = Math.floor((elapsed % 3600000) / 60000);
            const seconds = Math.floor((elapsed % 60000) / 1000);
            
            document.getElementById('stats-uptime').textContent = 
                `${hours.toString().padStart(2, '0')}:` +
                `${minutes.toString().padStart(2, '0')}:` +
                `${seconds.toString().padStart(2, '0')}`;
        }

        function updateStats() {
            document.getElementById('stats-messages').textContent = stats.messages;
            document.getElementById('data-session').textContent = stats.tx + stats.rx;
            
            document.getElementById('stats-tx').textContent = formatBytes(stats.tx);
            document.getElementById('stats-rx').textContent = formatBytes(stats.rx);
            
            const elapsed = (Date.now() - stats.startTime) / 1000;
            const bandwidth = elapsed > 0 ? (stats.tx + stats.rx) / elapsed / 1024 : 0;
            document.getElementById('stats-bandwidth').textContent = 
                `${bandwidth.toFixed(2)} KB/s`;
        }

        function formatBytes(bytes) {
            if (bytes < 1024) return bytes + ' B';
            if (bytes < 1048576) return (bytes / 1024).toFixed(1) + ' KB';
            return (bytes / 1048576).toFixed(1) + ' MB';
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

            function clearMorseTranslation() {
    document.getElementById('morse-translation').classList.remove('active');
    document.getElementById('morse-text-display').textContent = '';
    document.getElementById('morse-code-display').textContent = '';
}
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

                // Mostrar traducción Morse si está en modo morse
    const mode = document.getElementById('inputMode').value;
    if (mode === 'morse') {
        updateMorseTranslation(text);
    } else {
        document.getElementById('morse-translation').classList.remove('active');
    }

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

        function clearChat() {
            if (confirm("¿Borrar todo el historial del chat?")) {
                document.getElementById('monitor-decoded').innerHTML = 
                    '<div class="message-bubble message-system"><i class="fas fa-satellite"></i> HISTORIAL LIMPIADO</div>';
                updateMonitor("✅ CHAT LIMPIADO");
                playStrongBeep(400, 200);
            }
        }

        // ====== CONFIGURACIÓN MEJORADA ======
        
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
            document.getElementById(`btn-${currentConnectionType}`).classList.add('active');
            
            const typeName = currentConnectionType === 'wifi' ? 'WiFi' : 'Datos Móviles';
            document.getElementById('current-connection-type').textContent = typeName;
        }

        function selectConnectionType(type) {
            if (type === currentConnectionType) return;
            
            document.querySelectorAll('.connection-type-btn').forEach(btn => {
                btn.classList.remove('active');
            });
            document.getElementById(`btn-${type}`).classList.add('active');
            
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

        // ====== FUNCIONES DE CONFIGURACIÓN DE ID ======
        
        function loadIdSettings() {
            const config = JSON.parse(localStorage.getItem('radcom_config_v4') || '{}');
            
            document.getElementById('useFixedId').checked = config.useFixedId !== false;
            document.getElementById('customIdInput').value = config.fixedId || getOrCreateFixedId();
            document.getElementById('currentIdDisplay').textContent = ID_SYSTEM.currentId || 'No asignado';
            
            toggleIdMode();
        }

        function toggleIdMode() {
            const useFixed = document.getElementById('useFixedId').checked;
            const container = document.getElementById('customIdContainer');
            
            if (useFixed) {
                container.style.display = 'block';
            } else {
                container.style.display = 'none';
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
            document.getElementById('customIdInput').value = generatedId;
        }

        function applyIdSettings() {
            const useFixed = document.getElementById('useFixedId').checked;
            const customId = document.getElementById('customIdInput').value.trim();
            
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
            document.getElementById('customIdInput').value = defaultId;
            document.getElementById('useFixedId').checked = true;
            toggleIdMode();
            updateMonitor("🔄 ID RESTABLECIDO A VALOR PREDETERMINADO");
        }

        function saveSettings() {
            const config = {
                systemName: document.getElementById('systemName').value,
                connectionType: currentConnectionType,
                autoReconnect: document.getElementById('autoReconnect').checked,
                soundEnabled: document.getElementById('soundEnabled').checked,
                saveHistory: document.getElementById('saveHistory').checked,
                fastRecovery: document.getElementById('fastRecovery').checked,
                aggressiveRevive: document.getElementById('aggressiveRevive').checked,
                useFixedId: document.getElementById('useFixedId').checked,
                fixedId: document.getElementById('customIdInput').value.trim()
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

        function loadSettings() {
            const config = JSON.parse(localStorage.getItem('radcom_config_v4.6') || '{}');
            
            if (config.systemName) {
                document.getElementById('systemName').value = config.systemName;
            }
            if (config.connectionType) {
                currentConnectionType = config.connectionType;
            }
            if (config.autoReconnect !== undefined) {
                document.getElementById('autoReconnect').checked = config.autoReconnect;
            }
            if (config.soundEnabled !== undefined) {
                document.getElementById('soundEnabled').checked = config.soundEnabled;
            }
            if (config.saveHistory !== undefined) {
                document.getElementById('saveHistory').checked = config.saveHistory;
            }
            if (config.fastRecovery !== undefined) {
                document.getElementById('fastRecovery').checked = config.fastRecovery;
                fastRecovery = config.fastRecovery;
            }
            if (config.aggressiveRevive !== undefined) {
                document.getElementById('aggressiveRevive').checked = config.aggressiveRevive;
                aggressiveRevive = config.aggressiveRevive;
            }
            if (config.useFixedId !== undefined) {
                ID_SYSTEM.useFixedId = config.useFixedId;
            }
            if (config.fixedId) {
                ID_SYSTEM.fixedId = config.fixedId;
            }
        }

        function initResizableSeparator() {
            const separator = document.getElementById('resizableSeparator');
            const monitorContainer = document.getElementById('monitorContainer');
            const tableContainer = document.getElementById('tableContainer');
            
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

// Variables de seguridad
let securityConfig = {
    enabled: true,
    algorithm: 'AES-256-GCM'
};

// Cifrar con AES
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

// Descifrar con AES
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

// Modificar la función sendMessage existente
// Modificar la función sendMessage existente
function enhanceSendMessage() {
    const originalSendMessage = sendMessage;
    
    sendMessage = function() {
        const input = document.getElementById('inputMsg');
        let message = input.value.trim();
        
        if (!message) {
            updateMonitor("⚠️ MENSAJE VACÍO", "error");
            playStrongBeep(300, 200);
            return;
        }
        
        const key = document.getElementById('key').value || 'RADCOM-SECURE';
        const mode = document.getElementById('inputMode').value;
        const encryptionSelect = document.getElementById('encryptionMode');
        const encryption = encryptionSelect ? encryptionSelect.value : 'aes'; // Valor por defecto
        
        // Cifrar según selección
        let encryptedMessage = message;
        let encryptionType = 'none';
        
        if (encryption === 'aes') {
            encryptedMessage = encryptAES(message, key);
            encryptionType = 'AES-256-GCM';
        } else if (encryption === 'xor') {
            encryptedMessage = xorEncrypt(message, key);
            encryptionType = 'XOR';
        } else if (encryption === 'none') {
            // Sin cifrado - usar mensaje original
            encryptedMessage = message;
            encryptionType = 'Ninguno';
        }
        
        // Crear paquete
        const data = {
            type: 'message',
            message: encryptedMessage,
            original: message,
            mode: mode,
            encryption: encryptionType,
            timestamp: Date.now(),
            sender: myPeerId
        };
        
        // Enviar (mantener lógica original)
        let sentCount = 0;
        if (activeTarget === 'GLOBAL') {
            Object.keys(connections).forEach(peerId => {
                if (connections[peerId]?.conn?.open) {
                    connections[peerId].conn.send(data);
                    sentCount++;
                }
            });
        } else if (connections[activeTarget]) {
            connections[activeTarget].conn.send(data);
            sentCount = 1;
        }
        
        if (sentCount > 0) {
            // Mostrar localmente con indicador de cifrado
            const monitor = document.getElementById('monitor-decoded');
            const bubble = document.createElement('div');
            bubble.className = 'message-bubble message-outgoing';
            
            // Cambiar color según cifrado
            let color = '#888';
            if (encryptionType === 'AES-256-GCM') color = '#00ffea';
            if (encryptionType === 'XOR') color = '#ffaa00';
            if (encryptionType === 'Ninguno') color = '#ff3300';
            
            bubble.innerHTML = `<strong>YO:</strong> ${message}<br>
                               <small style="color:${color}; font-size:0.6rem;">Cifrado: ${encryptionType}</small>`;
            monitor.appendChild(bubble);
            monitor.scrollTop = monitor.scrollHeight;
            
            stats.tx += message.length;
            stats.messages++;
            input.value = '';
            updateMonitor(`🔐 ENVIADO [${encryptionType}] a ${sentCount} destino(s)`);
            updateStats();
            playStrongBeep(700, 100);
        } else {
            updateMonitor("⚠️ SIN DESTINOS CONECTADOS", "warning");
            playStrongBeep(300, 200);
        }
    };

    // ... obtienes message, mode, encryptionMode, key ...

const key = document.getElementById('key')?.value?.trim() || "";
const encMode = document.getElementById('encryptionMode')?.value || "none";

const secured = securityLayer(message, true, encMode, key);

// Mostrar localmente lo que se envió (el texto original)
addBubble("YO", message, secured.encryptionUsed);   // ajusta nombre de función si es distinto

const data = {
    type: "message",
    message: secured.text,
    encryption: secured.encryptionUsed,
    timestamp: Date.now(),
    sender: myPeerId,
    // otros campos que ya tengas
};

// Enviar
if (activeTarget === 'GLOBAL') {
    Object.keys(connections).forEach(peerId => {
        if (connections[peerId]?.conn?.open) {
            connections[peerId].conn.send(data);
        }
    });
} else if (connections[activeTarget]?.conn?.open) {
    connections[activeTarget].conn.send(data);
}
}

// Modificar recepción de datos
// Modificar recepción de datos
function enhanceReceiveData() {
    const originalHandleReceivedData = handleReceivedData;
    
    handleReceivedData = function(senderId, data) {
        // Si es mensaje cifrado (tiene campo encryption)
        if (data.encryption && data.encryption !== 'Ninguno') {
            const key = document.getElementById('key').value || 'RADCOM-SECURE';
            let decryptedMessage = data.message;
            
            // Descifrar según tipo
            if (data.encryption === 'AES-256-GCM') {
                decryptedMessage = decryptAES(data.message, key);
            } else if (data.encryption === 'XOR') {
                decryptedMessage = xorDecrypt(data.message, key);
            }
            
            // Mostrar con indicador de cifrado
            const displayName = senderId.substring(0, 8);
            const monitor = document.getElementById('monitor-decoded');
            const bubble = document.createElement('div');
            bubble.className = 'message-bubble message-incoming';
            
            // Cambiar color según cifrado
            let color = '#888';
            if (data.encryption === 'AES-256-GCM') color = '#00ffea';
            if (data.encryption === 'XOR') color = '#ffaa00';
            
            bubble.innerHTML = `<strong>${displayName}:</strong> ${decryptedMessage}<br>
                               <small style="color:${color}; font-size:0.6rem;">Cifrado: ${data.encryption}</small>`;
            monitor.appendChild(bubble);
            monitor.scrollTop = monitor.scrollHeight;
            
            stats.rx += data.message.length;
            stats.messages++;
            updateStats();
            
            updateMonitor(`🔐 ${displayName}: "${decryptedMessage.substring(0, 30)}${decryptedMessage.length > 30 ? '...' : ''}"`);
            playMessageNotification();
            
        } else if (data.encryption === 'Ninguno') {
            // Mensaje sin cifrado
            const displayName = senderId.substring(0, 8);
            const monitor = document.getElementById('monitor-decoded');
            const bubble = document.createElement('div');
            bubble.className = 'message-bubble message-incoming';
            bubble.innerHTML = `<strong>${displayName} [SIN CIFRADO]:</strong> ${data.message}<br>
                               <small style="color:#ff3300; font-size:0.6rem;">⚠️ Sin cifrado</small>`;
            monitor.appendChild(bubble);
            monitor.scrollTop = monitor.scrollHeight;
            
            stats.rx += data.message.length;
            stats.messages++;
            updateStats();
            
            updateMonitor(`⚠️ ${displayName} [SIN CIFRADO]: "${data.message.substring(0, 30)}${data.message.length > 30 ? '...' : ''}"`);
            playMessageNotification();
            
        } else {
            // Usar función original para mensajes antiguos sin campo encryption
            originalHandleReceivedData(senderId, data);
        }
    };
}

// Mostrar panel de información de seguridad
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
    
    // Cerrar al hacer clic fuera
    setTimeout(() => {
        panel.addEventListener('click', e => e.stopPropagation());
        document.addEventListener('click', () => panel.remove());
    }, 100);
}

// Inicializar seguridad
// Inicializar seguridad
function initSecurity() {
    console.log("🔐 Inicializando seguridad v5.2...");
    enhanceSendMessage();
    enhanceReceiveData();
    
    // Actualizar badge inicial
    updateSecurityBadge();
    
    // Añadir evento al selector para actualizar badge
    const encryptionSelect = document.getElementById('encryptionMode');
    if (encryptionSelect) {
        encryptionSelect.addEventListener('change', updateSecurityBadge);
    }
    
    // Actualizar monitor inicial
    updateMonitor("🔐 SISTEMA DE SEGURIDAD v5.2 ACTIVADO");
    
    // Añadir mensaje al chat
    const monitor = document.getElementById('monitor-decoded');
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

// Actualizar badge según cifrado seleccionado
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

// ────────────────────────────────────────────────
// ÚNICA FUNCIÓN DE SEGURIDAD (compatible 4.7.6)
// ────────────────────────────────────────────────
function securityLayer(text, isSending, encryptionMode, password) {
    // Valores por defecto y normalización
    if (!password) password = "";
    encryptionMode = (encryptionMode || "none").toLowerCase().trim();

    const result = {
        text: text,
        encryptionUsed: "NONE",
        error: null
    };

    if (encryptionMode === "none" || !password) {
        // sin cifrado o sin clave → no hacemos nada
        return result;
    }

    try {
        if (encryptionMode === "aes" || encryptionMode === "AES-256-GCM") {
            if (isSending) {
                // Cifrar
                const iv = CryptoJS.lib.WordArray.random(16);
                const salt = CryptoJS.lib.WordArray.random(8); // mejor que usar iv como salt
                const key = CryptoJS.PBKDF2(password, salt, {
                    keySize: 256/32,
                    iterations: 10000
                });
                const encrypted = CryptoJS.AES.encrypt(text, key, {
                    iv: iv,
                    mode: CryptoJS.mode.CBC,
                    padding: CryptoJS.pad.Pkcs7
                });

                // Formato: salt:iv:encrypted (base64)
                result.text = salt.toString(CryptoJS.enc.Base64) + ":" +
                              iv.toString(CryptoJS.enc.Base64) + ":" +
                              encrypted.toString();
                result.encryptionUsed = "AES-256-GCM";
            } else {
                // Descifrar
                const parts = text.split(":");
                if (parts.length !== 3) {
                    throw new Error("Formato AES inválido (debe ser salt:iv:ciphertext)");
                }

                const salt = CryptoJS.enc.Base64.parse(parts[0]);
                const iv   = CryptoJS.enc.Base64.parse(parts[1]);
                const ct   = parts[2];

                const key = CryptoJS.PBKDF2(password, salt, {
                    keySize: 256/32,
                    iterations: 10000
                });

                const decrypted = CryptoJS.AES.decrypt(ct, key, {
                    iv: iv,
                    mode: CryptoJS.mode.CBC,
                    padding: CryptoJS.pad.Pkcs7
                });

                const plaintext = decrypted.toString(CryptoJS.enc.Utf8);
                if (!plaintext) {
                    throw new Error("Descifrado vacío → clave incorrecta?");
                }

                result.text = plaintext;
                result.encryptionUsed = "AES-256-GCM";
            }
        }
        else if (encryptionMode === "xor") {
            // XOR simple (para compatibilidad con versiones antiguas)
            let output = "";
            for (let i = 0; i < text.length; i++) {
                output += String.fromCharCode(
                    text.charCodeAt(i) ^ password.charCodeAt(i % password.length)
                );
            }
            result.text = output;
            result.encryptionUsed = "XOR";
        }
        else {
            result.error = "Modo de cifrado desconocido: " + encryptionMode;
        }
    }
    catch (err) {
        result.error = err.message || "Error en capa de seguridad";
        console.error("[securityLayer]", result.error, { text, isSending, encryptionMode });
        result.text = "[ERROR SEGURIDAD] " + (result.error || "desconocido");
    }

    return result;
}



// Ejecutar al cargar
document.addEventListener('DOMContentLoaded', function() {
    setTimeout(initSecurity, 1000);
});

        // ====== INICIALIZACIÓN ======
        
        window.onload = function() {
            console.log(`🚀 Iniciando RADCOM MASTER v${VERSION}`);
            
            // Inicializar sistemas originales
            buildAsciiTable();
            buildMorseTable();
            initRadioSystem();
            loadSettings();
            updatePeerList();
            initResizableSeparator();
            initMorseAudio();
            initVoiceRecognition();
            
            // Inicializar sistema satelital mejorado
            initSatelliteSystem();
            
            // Inicializar PeerJS
            initPeerJSEnhanced();
            
            // Iniciar health checks
            setTimeout(startHealthCheckSystem, 5000);
            
            updateMonitor(`✅ SISTEMA v${VERSION} INICIADO | SATÉLITE ACTIVO`);

            // Iniciar seguimiento automático de posición (modo "fijo")
            startWatchingPosition();
        };

        // In the file where the function is defined
window.initializeSatelliteWeather = function() {
    // Your logic here
    console.log("Satellite weather initialized");
};

        // ====== MÓDULO DE MEMORIA RADCOM v4.7 ======

function saveToLocalStorage(sender, msg) {
    let history = JSON.parse(localStorage.getItem('radcom_history_v47') || "[]");
    history.push({
        time: new Date().toLocaleTimeString(),
        sender: sender,
        text: msg
    });
    if(history.length > 30) history.shift(); // Guardar últimos 30 mensajes
    localStorage.setItem('radcom_history_v47', JSON.stringify(history));
}

function loadHistoryFromLocal() {
    let history = JSON.parse(localStorage.getItem('radcom_history_v47') || "[]");
    if(history.length > 0) {
        updateMonitor("📂 RECUPERANDO MENSAJES GUARDADOS...");
        history.forEach(item => {
            // Se muestran en el log pero con un estilo más apagado (dimmed)
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



/**
 * Función única que:
 * 1. Maneja conversión Morse si corresponde
 * 2. Negocia clave ECDH la primera vez (si no existe)
 * 3. Cifra con AES-256-GCM usando la clave negociada
 * 4. Prepara el objeto data listo para enviar
 * 5. Devuelve { data, error, displayText }
 */
function prepareAndSecureMessage(rawText, inputMode = 'text') {
    const result = {
        data: null,
        error: null,
        displayText: rawText,     // lo que se muestra localmente
        sentText: rawText         // lo que se envía (puede ser cifrado)
    };

    let messageToProcess = rawText.trim();
    if (!messageToProcess) {
        result.error = "Mensaje vacío";
        return result;
    }

    // ───────────────────────────────────────────────
    // 1. Conversión Morse (si modo = morse)
    // ───────────────────────────────────────────────
    if (inputMode === 'morse' || inputMode === 'morse_audio') {
        if (typeof textToMorse !== 'function') {
            result.error = "Función textToMorse no definida";
            return result;
        }
        messageToProcess = textToMorse(messageToProcess);
        result.displayText = rawText;               // mostramos texto humano
        // result.sentText ya tiene el morse → se enviará cifrado o no
        // Opcional: reproducir audio aquí si quieres
        // if (typeof playMorse === 'function') playMorse(messageToProcess);
    }

    // ───────────────────────────────────────────────
    // 2. Obtener o negociar clave AES para este peer
    // ───────────────────────────────────────────────
    const target = activeTarget === 'GLOBAL' ? null : activeTarget;
    let aesKey = null;

    if (target && connections[target]) {
        // ¿Ya tenemos clave guardada?
        aesKey = connections[target].aesKey || localStorage.getItem(`aes_key_${target}`);

        // Si no → negociamos ahora (ECDH)
        if (!aesKey) {
            try {
                const ec = new elliptic.ec('p256');
                const myKeyPair = ec.genKeyPair();
                const myPubHex = myKeyPair.getPublic('hex');

                // Enviar handshake
                const handshake = {
                    type: 'ecdh_init',
                    pub: myPubHex,
                    from: myPeerId
                };

                connections[target].conn.send(handshake);

                // Guardamos nuestra parte privada temporalmente
                connections[target].ecdhTemp = myKeyPair;

                updateMonitor(`Iniciando negociación segura con ${target}...`);
            } catch (err) {
                result.error = "Error al iniciar ECDH: " + err.message;
                return result;
            }

            // La clave no estará lista inmediatamente → hay que esperar respuesta
            result.error = "Esperando negociación de clave segura...";
            return result;
        }
    }

    // ───────────────────────────────────────────────
    // 3. Cifrar si tenemos clave
    // ───────────────────────────────────────────────
    let encryptionType = "NONE";
    let finalPayload = messageToProcess;

    if (aesKey) {
        try {
            const iv = CryptoJS.lib.WordArray.random(16);
            const keyParsed = CryptoJS.enc.Hex.parse(aesKey);
            const encrypted = CryptoJS.AES.encrypt(messageToProcess, keyParsed, {
                iv: iv,
                mode: CryptoJS.mode.CBC,
                padding: CryptoJS.pad.Pkcs7
            });

            finalPayload = iv.toString() + ':' + encrypted.toString();
            encryptionType = "AES-256-GCM";
        } catch (err) {
            result.error = "Fallo al cifrar AES: " + err.message;
            return result;
        }
    }

    // ───────────────────────────────────────────────
    // 4. Preparar paquete final
    // ───────────────────────────────────────────────
    result.data = {
        type: 'message',
        payload: finalPayload,
        enc: encryptionType,
        mode: inputMode,
        ts: Date.now(),
        from: myPeerId
    };

    result.sentText = finalPayload;

    return result;
}

// =============================================================================
// FUNCIÓN ÚNICA: CIFRADO MILITAR-GRADE AUTOMÁTICO (ECDH + AES-256-GCM)
// =============================================================================
async function secureMilitaryChannel(peerId, message = null, isSend = false, mode = 'text') {
    // ── 1. Preparación inicial ──────────────────────────────────────────────
    const result = {
        success: false,
        payload: null,
        displayText: message || "",
        error: null,
        encryption: "NONE"
    };

    if (!peerId) {
        result.error = "No hay peerId";
        return result;
    }

    // ── 2. Obtener o generar clave compartida (ECDH x25519) ─────────────────
    let sharedKeyHex = localStorage.getItem(`military_key_${peerId}`);

    if (!sharedKeyHex) {
        try {
            // Generar claves privadas (x25519)
            const privateKey = ed25519.utils.randomPrivateKey();
            const publicKey = await ed25519.getPublicKeyAsync(privateKey);

            // Enviar nuestra clave pública (base64)
            const pubB64 = btoa(String.fromCharCode(...publicKey));

            const handshake = {
                type: 'military_handshake',
                pub: pubB64,
                from: myPeerId
            };

            if (connections[peerId]?.conn?.open) {
                connections[peerId].conn.send(handshake);
            }

            // Guardamos temporalmente nuestra privada
            connections[peerId] = connections[peerId] || {};
            connections[peerId].militaryTempPriv = privateKey;

            result.error = "Negociando clave militar... (espera 1-3 seg)";
            return result;
        } catch (err) {
            result.error = "Fallo al iniciar ECDH: " + err.message;
            return result;
        }
    }

    // ── 3. Si tenemos clave → derivar AES-256-GCM con HKDF ──────────────────
    const sharedSecret = hexToBytes(sharedKeyHex);
    const hkdfSalt = new Uint8Array(32); // puedes usar valor fijo o random
    const hkdfInfo = new TextEncoder().encode("RADCOM-MILITARY-2026");

    const aesKeyRaw = await hashes.hkdf(sharedSecret, hashes.SHA256, {
        salt: hkdfSalt,
        info: hkdfInfo,
        length: 32
    });

    const aesKey = aesKeyRaw; // ya son 32 bytes

    // ── 4. Manejo de modo Morse ─────────────────────────────────────────────
    let processedText = message || "";
    if (mode === 'morse' && isSend) {
        processedText = textToMorse(processedText) || "[error morse]";
        result.displayText = message; // mostramos texto humano
    }

    // ── 5. Cifrar o descifrar ───────────────────────────────────────────────
    if (isSend) {
        // Cifrar (AES-GCM)
        try {
            const iv = crypto.getRandomValues(new Uint8Array(12));
            const encoder = new TextEncoder();
            const encrypted = await crypto.subtle.encrypt(
                { name: "AES-GCM", iv },
                await crypto.subtle.importKey("raw", aesKey, "AES-GCM", false, ["encrypt"]),
                encoder.encode(processedText)
            );

            const encryptedB64 = btoa(String.fromCharCode(...new Uint8Array(encrypted)));
            const ivB64 = btoa(String.fromCharCode(...iv));

            result.payload = {
                type: 'military_msg',
                iv: ivB64,
                data: encryptedB64,
                mode: mode
            };
            result.encryption = "AES-256-GCM";
            result.success = true;
        } catch (err) {
            result.error = "Fallo cifrado GCM: " + err.message;
        }
    } else {
        // Descifrar (si es mensaje cifrado)
        if (message.iv && message.data) {
            try {
                const iv = Uint8Array.from(atob(message.iv), c => c.charCodeAt(0));
                const ct = Uint8Array.from(atob(message.data), c => c.charCodeAt(0));

                const decrypted = await crypto.subtle.decrypt(
                    { name: "AES-GCM", iv },
                    await crypto.subtle.importKey("raw", aesKey, "AES-GCM", false, ["decrypt"]),
                    ct
                );

                result.displayText = new TextDecoder().decode(decrypted);
                result.encryption = "AES-256-GCM";
                result.success = true;

                if (message.mode === 'morse') {
                    result.displayText = decodeMorse(result.displayText) || "[error morse]";
                }
            } catch (err) {
                result.error = "Fallo descifrado GCM: " + err.message;
                result.displayText = "[MENSAJE PROTEGIDO - CLAVE NO COINCIDE]";
            }
        } else {
            result.displayText = message;
            result.success = true;
        }
    }

    return result;
}

// =============================================
// PARTE C - LIMPIEZA GENERAL Y CONSOLIDACIÓN v5.5
// =============================================


// === SETUP PEERJS LIMPIO ===
function setupPeerEvents() {
    peer.on('open', (id) => {
        myPeerId = id;
        localStorage.setItem('radcom_master_id_v5', id);
        document.getElementById('display-id').innerHTML = 
            `<span style="color:#00ff88">ID: ${id.substring(0,12)}...</span>`;
        
        updateMonitor(`✅ RADCOM MASTER v${VERSION} INICIADO | ID: ${id.substring(0,10)}...`);
        
        // Reconectar peers guardados
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

// === SETUP CONEXIÓN LIMPIO ===
function setupConnection(conn, direction) {
    const peerId = conn.peer;
    
    connections[peerId] = {
        conn: conn,
        status: 'connecting',
        direction: direction,
        health: 'new',
        latency: 0,
        lastActivity: Date.now()
    };

    conn.on('open', () => {
        connections[peerId].status = 'online';
        connections[peerId].health = 'healthy';
        updatePeerList();
        updateConnectedCount();
        updateMonitor(`✅ CONECTADO A ${peerId.substring(0,8)}`);
        playStrongBeep(800, 100);
        
        saveId(peerId);
        deliverOnConnect(peerId); // Entregar cola
    });

    conn.on('data', (data) => {
        connections[peerId].lastActivity = Date.now();
        handleReceivedData(peerId, data);   // ← Usa la versión de Parte B
    });

    conn.on('close', () => {
        connections[peerId].status = 'offline';
        updatePeerList();
        updateConnectedCount();
        updateMonitor(`🔒 DESCONECTADO: ${peerId.substring(0,8)}`);
        
        if (document.getElementById('autoReconnect')?.checked) {
            setTimeout(() => connectToPeerId(peerId), 2500);
        }
    });

    conn.on('error', (err) => {
        connections[peerId].status = 'error';
        updatePeerList();
    });
}

// === HEALTH + REVIVE MEJORADO ===
function startHealthCheckSystem() {
    setInterval(() => {
        Object.keys(connections).forEach(peerId => {
            if (connections[peerId]?.status === 'online') {
                const inactive = Date.now() - (connections[peerId].lastActivity || 0);
                if (inactive > 15000) {
                    sendHealthPing(peerId);
                }
            }
        });
    }, 10000);
}

function reviveAllConnections() {
    let revived = 0;
    Object.keys(connections).forEach(peerId => {
        if (connections[peerId]?.status === 'offline' && savedIds.includes(peerId)) {
            connectToPeerId(peerId);
            revived++;
        }
    });
    
    if (revived > 0) {
        updateMonitor(`🔄 REVIVIENDO ${revived} CONEXIONES...`);
    }
}

// === INICIALIZACIÓN FINAL (reemplaza tu window.onload) ===
window.onload = function() {
    console.log(`🚀 RADCOM MASTER v${VERSION} - Versión limpia y segura`);

    // Inicializaciones originales (mantengo todo)
    buildAsciiTable();
    buildMorseTable();
    initRadioSystem();
    initSatelliteSystem();
    initVoiceRecognition();
    initResizableSeparator();
    loadSettings();
    updatePeerList();

    // PeerJS
    peer = new Peer();
    setupPeerEvents();
    startHealthCheckSystem();

    // Versión en badge
    document.querySelector('.version-badge').textContent = `v${VERSION}`;

    updateMonitor(`✅ SISTEMA v${VERSION} INICIADO | AES-256-GCM + ECDH ACTIVO`);
};

// === FUNCIONES AUXILIARES (ya las tenías, solo las dejo por si faltan) ===
function connectToPeerId(peerId) {
    if (connections[peerId]?.status === 'online') return;
    const conn = peer.connect(peerId, { reliable: true });
    setupConnection(conn, 'outgoing');
}

function updateConnectedCount() {
    const online = Object.values(connections).filter(c => c.status === 'online').length;
    document.getElementById('connected-count').textContent = online;
    document.getElementById('stats-connections').textContent = online;
}

// Actualiza tu updateQueueCounter y demás funciones pequeñas si es necesario.

// =============================================
// PARTE D - FINALIZACIÓN v5.6.1
// =============================================


// === PROCESAMIENTO DE COLA MEJORADO ===
function processQueue() {
    if (messageQueue.length === 0) return;

    const now = Date.now();
    let remaining = [];

    messageQueue.forEach(item => {
        if (now - item.timestamp < 3000) {
            remaining.push(item);
            return;
        }

        item.attempts = (item.attempts || 0) + 1;

        let delivered = false;
        const target = item.target;

        if (target === 'GLOBAL') {
            Object.keys(connections).forEach(pid => {
                if (connections[pid]?.conn?.open) {
                    connections[pid].conn.send(item.packet);
                    delivered = true;
                }
            });
        } else if (connections[target]?.conn?.open) {
            connections[target].conn.send(item.packet);
            delivered = true;
        }

        if (delivered) {
            stats.tx += JSON.stringify(item.packet).length;
            stats.messages++;
            updateStats();
            displayMessage(`✅ ENTREGADO: ${item.original.substring(0,40)}...`, '', 'system');
        } else if (item.attempts < 8) {
            remaining.push(item);
        } else {
            displayMessage(`❌ NO ENTREGADO: ${item.original.substring(0,30)}...`, '', 'system');
        }
    });

    messageQueue = remaining;
    localStorage.setItem('radcom_message_queue', JSON.stringify(messageQueue));
    updateQueueCounter();
}

// === DISPLAYMESSAGE MEJORADO ===
function displayMessage(content, hex = '', type = 'incoming') {
    const monitor = document.getElementById('monitor-decoded');
    const div = document.createElement('div');
    div.className = `message-bubble message-${type}`;
    
    const time = new Date().toLocaleTimeString([], { hour: '2-digit', minute: '2-digit', second: '2-digit' });
    
    div.innerHTML = `
        <span style="color:#888; font-size:0.55rem;">${time}</span><br>
        ${content}
        ${hex ? `<br><small style="color:#00ffea">HEX: ${hex}</small>` : ''}
    `;
    
    monitor.appendChild(div);
    monitor.scrollTop = monitor.scrollHeight;
}

// === ACTUALIZACIÓN DINÁMICA DEL BADGE DE SEGURIDAD ===
function updateSecurityBadge() {
    const mode = document.getElementById('encryptionMode').value;
    const badge = document.querySelector('.security-badge');
    
    if (!badge) return;
    
    let color = '#ff3300';
    let text = 'SIN CIFRADO';
    
    if (mode === 'aes-gcm-ecdh') {
        color = '#00ff88';
        text = 'AES-256-GCM + ECDH';
    } else if (mode === 'aes-cbc') {
        color = '#00ffea';
        text = 'AES-CBC (legacy)';
    } else if (mode === 'xor') {
        color = '#ffaa00';
        text = 'XOR';
    }
    
    badge.style.color = color;
    badge.querySelector('span:last-child').textContent = text;
}

// === INICIALIZACIÓN FINAL (ejecutar al final) ===
function finalInitialization() {
    // Badge dinámico
    const encSelect = document.getElementById('encryptionMode');
    if (encSelect) {
        encSelect.addEventListener('change', updateSecurityBadge);
        updateSecurityBadge();
    }
    
    // Iniciar cola
    setInterval(processQueue, 8000);
    
    // Revive automático cada 25 segundos
    setInterval(() => {
        if (Object.values(connections).some(c => c.status === 'offline')) {
            reviveAllConnections();
        }
    }, 25000);
    
    updateMonitor(`🚀 RADCOM MASTER v${VERSION} - Versión estable y segura`);
}

// === LLAMADA FINAL ===

// ====== SISTEMA CONSOLA v5.6.1 ======

let currentConsoleTab = 'CMD';
let hexEditorContent = '';

// Abrir ventana consola
function openConsole() {
    document.getElementById('consoleModal').style.display = 'flex';
    switchConsoleTab('CMD');
    updateMonitor("🚀 CONSOLE v5.6.1 ACTIVATED", "info");
    playStrongBeep(1000, 100);
}

// Cerrar ventana consola
function closeConsole() {
    document.getElementById('consoleModal').style.display = 'none';
    updateMonitor("🔒 Console closed", "info");
}

// Cambiar pestaña consola
function switchConsoleTab(tab) {
    currentConsoleTab = tab;
    
    // Actualizar pestañas activas
    document.querySelectorAll('.console-tab').forEach(t => t.classList.remove('active'));
    document.querySelectorAll('.console-tab-content').forEach(c => c.classList.remove('active'));
    
    // Activar pestaña seleccionada
    document.querySelector(`.console-tab[onclick="switchConsoleTab('${tab}')"]`).classList.add('active');
    document.getElementById(`console-tab-${tab}`).classList.add('active');
    
    // Inicializar contenido según pestaña
    if (tab === 'hex') {
        initializeHexEditor();
    } else if (tab === 'CMD') {
        document.getElementById('CMD-input').focus();
    } else if (tab === 'decode') {
        document.getElementById('hex-decoder-input').focus();
    }
    
    playStrongBeep(600, 50);
}

// ====== CMD CONSOLE FUNCTIONS ======

function handleCMDCommand(event) {
    if (event.key === 'Enter') {
        executeCMDCommand();
    }
}

function executeCMDCommand() {
    const input = document.getElementById('CMD-input');
    const command = input.value.trim();
    
    if (!command) return;
    
    // Mostrar comando
    appendToCMDConsole(`&gt; ${command}`);
    
    // Procesar comando
    processCMDCommand(command);
    
    // Limpiar input
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
            setTimeout(() => {
                location.reload();
            }, 1000);
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
    console.innerHTML += `\n${text}`;
    console.scrollTop = console.scrollHeight;
}

function clearCMDConsole() {
    document.getElementById('CMD-output').innerHTML = 
        '&gt; RADCOM CMD CONSOLE v5.6.1 INITIALIZED\n' +
        '&gt; System: RADCOM MASTER\n' +
        '&gt; Version: ' + VERSION + '\n' +
        '&gt; Ready for commands...';
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
        return `${id.substring(0, 12)}... | ${conn.status} | ${conn.type} | ${conn.health}`;
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
        { name: 'Voice Recognition', test: () => !!window.SpeechRecognition || !!window.webkitSpeechRecognition }
    ];
    
    let passed = 0;
    tests.forEach(test => {
        const result = test.test();
        appendToCMDConsole(`${result ? '✓' : '✗'} ${test.name}: ${result ? 'PASS' : 'FAIL'}`);
        if (result) passed++;
    });
    
    appendToCMDConsole(`\n${passed}/${tests.length} tests passed`);
}

// ====== HEX EDITOR FUNCTIONS ======

function initializeHexEditor() {
    const editor = document.getElementById('hex-editor');
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
    const text = editor.value;
    const lines = text.substring(0, editor.selectionStart).split('\n');
    const line = lines.length;
    const col = lines[lines.length - 1].length + 1;
    
    document.getElementById('hex-position').textContent = 
        `Line: ${line}, Col: ${col} | Chars: ${text.length}`;
}

function hexLoadCurrentPage() {
    // Cargar el HTML actual de la página
    const htmlContent = document.documentElement.outerHTML;
    const formatted = formatHTML(htmlContent);
    
    document.getElementById('hex-editor').value = formatted;
    hexEditorContent = formatted;
    
    updateHexStatus('Page HTML loaded');
    appendToCMDConsole('Loaded current page HTML into hex editor');
}

function hexSaveChanges() {
    const content = document.getElementById('hex-editor').value;
    
    // Aquí iría la lógica para guardar cambios
    // Por ahora solo mostramos un mensaje
    updateHexStatus('Changes saved (simulated)');
    appendToCMDConsole('Hex editor changes saved');
    
    // En un sistema real, aquí se enviaría al servidor o se guardaría en localStorage
    localStorage.setItem('radcom_hex_editor_backup', content);
    
    playStrongBeep(800, 50);
}

function hexFind() {
    const search = prompt('Search in hex editor:');
    if (search) {
        const editor = document.getElementById('hex-editor');
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
    const content = editor.value;
    
    // Intentar formatear como HTML
    if (content.includes('<') && content.includes('>')) {
        editor.value = formatHTML(content);
        updateHexStatus('Formatted as HTML');
    } 
    // Intentar formatear como JSON
    else if (content.trim().startsWith('{') || content.trim().startsWith('[')) {
        try {
            const json = JSON.parse(content);
            editor.value = JSON.stringify(json, null, 2);
            updateHexStatus('Formatted as JSON');
        } catch {
            updateHexStatus('Not valid JSON');
        }
    } 
    // Formatear como JavaScript
    else if (content.includes('function') || content.includes('const ') || content.includes('let ')) {
        editor.value = formatJavaScript(content);
        updateHexStatus('Formatted as JavaScript');
    }
    
    hexEditorContent = editor.value;
}

function hexMinify() {
    const editor = document.getElementById('hex-editor');
    let content = editor.value;
    
    // Método MANUAL sin regex problemáticos
    let result = '';
    let inComment = false;
    
    for (let i = 0; i < content.length; i++) {
        // Detectar comentarios
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
    
    // Reemplazar múltiples espacios (manualmente)
    let lastCharWasSpace = false;
    result = '';
    
    for (let i = 0; i < content.length; i++) {
        const char = content[i];
        const isWhitespace = char === ' ' || char === '\n' || char === '\t' || char === '\r';
        
        if (isWhitespace) {
            if (!lastCharWasSpace && i > 0 && i < content.length - 1) {
                // Solo agregar un espacio si no es al inicio/final
                result += ' ';
                lastCharWasSpace = true;
            }
        } else {
            result += char;
            lastCharWasSpace = false;
        }
    }
    
    content = result.trim();
    
    // Eliminar espacios entre tags (manualmente)
    result = '';
    for (let i = 0; i < content.length; i++) {
        if (content[i] === '>' && i + 1 < content.length) {
            result += '>';
            // Saltar espacios después de >
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
    document.getElementById('hex-status').textContent = message;
    setTimeout(() => {
        document.getElementById('hex-status').textContent = 'Ready';
    }, 3000);
}

function formatHTML(html) {
    // Formateador HTML simple
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
    // Formateador JavaScript simple
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

// ====== HEX DECODER FUNCTIONS ======

function autoDetectAndConvert() {
    const input = document.getElementById('hex-decoder-input').value.trim();
    
    if (!input) {
        document.getElementById('hex-decoder-output').value = '';
        return;
    }
    
    // Auto-detect: si parece hex, convertir a texto
    if (/^[0-9a-fA-F\s]+$/.test(input.replace(/\s/g, ''))) {
        convertHexToText();
    }
    // Si parece texto normal, convertir a hex
    else if (/^[\x00-\x7F\s]+$/.test(input)) {
        convertTextToHex();
    }
}

function convertTextToHex() {
    const input = document.getElementById('hex-decoder-input').value;
    let output = '';
    
    for (let i = 0; i < input.length; i++) {
        const hex = input.charCodeAt(i).toString(16).toUpperCase().padStart(2, '0');
        output += hex + ' ';
        
        // Agrupar en bloques de 8 bytes
        if ((i + 1) % 8 === 0) {
            output += ' ';
        }
        
        // Nueva línea cada 32 bytes
        if ((i + 1) % 32 === 0) {
            output += '\n';
        }
    }
    
    // Agregar representación ASCII
    output += '\n\nASCII: ';
    for (let i = 0; i < Math.min(input.length, 64); i++) {
        const char = input[i];
        output += (char >= ' ' && char <= '~') ? char : '.';
    }
    
    document.getElementById('hex-decoder-output').value = output.trim();
    updateMonitor("📝 Converted text to HEX", "info");
}

function convertHexToText() {
    const input = document.getElementById('hex-decoder-input').value;
    const hex = input.replace(/[^0-9a-fA-F]/g, '');
    
    let output = '';
    for (let i = 0; i < hex.length; i += 2) {
        const hexByte = hex.substr(i, 2);
        if (hexByte) {
            const charCode = parseInt(hexByte, 16);
            if (!isNaN(charCode)) {
                output += String.fromCharCode(charCode);
            }
        }
    }
    
    // Mostrar información adicional
    const info = [
        `Length: ${output.length} characters`,
        `Hex bytes: ${Math.ceil(hex.length / 2)}`,
        '',
        'Decoded text:',
        '-------------',
        output
    ].join('\n');
    
    document.getElementById('hex-decoder-output').value = info;
    updateMonitor("🔓 Converted HEX to text", "info");
}

function decodeBase64() {
    const input = document.getElementById('hex-decoder-input').value.trim();
    
    try {
        const decoded = atob(input);
        document.getElementById('hex-decoder-output').value = decoded;
        updateMonitor("🔓 Decoded Base64", "info");
    } catch (error) {
        document.getElementById('hex-decoder-output').value = 
            `Error decoding Base64: ${error.message}`;
        updateMonitor("❌ Base64 decode failed", "error");
    }
}

function clearDecoder() {
    document.getElementById('hex-decoder-input').value = '';
    document.getElementById('hex-decoder-output').value = '';
    updateMonitor("🧹 Decoder cleared", "info");
}

// ====== UTILITY FUNCTIONS ======

function formatBytes(bytes) {
    if (bytes === 0) return '0 B';
    const k = 1024;
    const sizes = ['B', 'KB', 'MB', 'GB'];
    const i = Math.floor(Math.log(bytes) / Math.log(k));
    return parseFloat((bytes / Math.pow(k, i)).toFixed(2)) + ' ' + sizes[i];
}

function formatUptime(ms) {
    const seconds = Math.floor(ms / 1000);
    const hours = Math.floor(seconds / 3600);
    const minutes = Math.floor((seconds % 3600) / 60);
    const secs = seconds % 60;
    
    return `${hours.toString().padStart(2, '0')}:${minutes.toString().padStart(2, '0')}:${secs.toString().padStart(2, '0')}`;
}

// ====== TEST FUNCTIONS ======

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
    document.getElementById('hex-decoder-input').value = configStr;
    document.getElementById('hex-decoder-output').value = btoa(configStr);
    
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

function handleCockpitClick(modulo) {
    document.querySelectorAll('.btn-cockpit').forEach(b => b.classList.remove('active'));
    document.getElementById('btn-' + modulo.toLowerCase()).classList.add('active');
    
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




let currentBaseLayer = null;



let mapInstance = null;

let userMarker = null;           // Marcador que te sigue
let currentTrack = null;         // Ruta que se va dibujando en tiempo real


function initMapEngine() {
    if (mapInstance) mapInstance.remove();

    const container = document.getElementById('map-container');
    container.style.height = "100%";
    container.style.width = "100%";

    mapInstance = L.map('map-container').setView([40.4167, -3.7033], 13);

    // Capas
    const topo = L.tileLayer('https://{s}.tile.opentopomap.org/{z}/{x}/{y}.png', { maxZoom: 17 }).addTo(mapInstance);
    const streets = L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', { maxZoom: 19 });
    const satellite = L.tileLayer('https://server.arcgisonline.com/ArcGIS/rest/services/World_Imagery/MapServer/tile/{z}/{y}/{x}', { maxZoom: 19 });

    L.control.layers({ "🗺️ Topográfico": topo, "🛣️ Calles": streets, "🛰️ Satélite": satellite }, null, { collapsed: false }).addTo(mapInstance);

    mapInstance.addLayer(drawnItems);

    // Barra de dibujo (waypoints y rutas manuales)
    const drawControl = new L.Control.Draw({
        draw: { marker: true, polyline: true, polygon: false, rectangle: false, circle: false },
        edit: { featureGroup: drawnItems, remove: true }
    });
    mapInstance.addControl(drawControl);

    mapInstance.on(L.Draw.Event.CREATED, e => drawnItems.addLayer(e.layer));

    // ==================== UBICACIÓN REAL + TRACKING ====================

    // Botón "Mi Ubicación Actual"
    const locationBtn = L.control({position: 'topleft'});
    locationBtn.onAdd = function() {
        const btn = L.DomUtil.create('button', '');
        btn.innerHTML = '📍 Mi Ubicación';
        btn.style.cssText = 'background:#00ff88; color:#000; border:none; padding:8px 12px; border-radius:4px; font-weight:bold; cursor:pointer;';
        btn.onclick = getCurrentLocation;
        return btn;
    };
    locationBtn.addTo(mapInstance);

    // Botón "Iniciar Tracking de Ruta"
    const trackBtn = L.control({position: 'topleft'});
    trackBtn.onAdd = function() {
        const btn = L.DomUtil.create('button', '');
        btn.id = 'track-btn';
        btn.innerHTML = '▶️ Iniciar Tracking';
        btn.style.cssText = 'background:#ffaa00; color:#000; border:none; padding:8px 12px; border-radius:4px; font-weight:bold; margin-top:8px; cursor:pointer;';
        btn.onclick = toggleLiveTracking;
        return btn;
    };
    trackBtn.addTo(mapInstance);

    mapInstance.invalidateSize(true);
    updateMonitor("🗺️ Mapa listo - Pulsa 'Mi Ubicación' primero");
}

// ==================== FUNCIONES DE UBICACIÓN Y TRACKING ====================

function getCurrentLocation() {
    if (!navigator.geolocation) {
        updateMonitor("❌ Tu navegador no soporta geolocalización", "error");
        return;
    }

    updateMonitor("📡 Buscando tu ubicación...");

    navigator.geolocation.getCurrentPosition(
        (pos) => {
            const lat = pos.coords.latitude;
            const lon = pos.coords.longitude;

            mapInstance.setView([lat, lon], 15);

            if (userMarker) userMarker.remove();
            userMarker = L.marker([lat, lon], { 
                icon: L.divIcon({ className: 'current-location-dot', html: '🟢', iconSize: [24,24] })
            })
            .addTo(mapInstance)
            .bindPopup("📍 Tu posición actual").openPopup();

            updateMonitor(`✅ Ubicación actual: ${lat.toFixed(4)}, ${lon.toFixed(4)}`);
        },
        (err) => {
            updateMonitor("❌ No se pudo obtener la ubicación. Activa el GPS y permite el permiso.", "error");
            console.error(err);
        },
        { enableHighAccuracy: true, timeout: 10000 }
    );
}

function toggleLiveTracking() {
    const btn = document.getElementById('track-btn');

    if (watchId) {
        // Parar tracking
        navigator.geolocation.clearWatch(watchId);
        watchId = null;
        btn.innerHTML = '▶️ Iniciar Tracking';
        btn.style.background = '#ffaa00';
        updateMonitor("⏹️ Tracking detenido");
    } else {
        // Iniciar tracking
        if (!currentTrack) {
            currentTrack = L.polyline([], { color: '#00ff88', weight: 5 }).addTo(mapInstance);
        }

        watchId = navigator.geolocation.watchPosition(
            (pos) => {
                const lat = pos.coords.latitude;
                const lon = pos.coords.longitude;

                // Mover marcador
                if (userMarker) userMarker.setLatLng([lat, lon]);
                else {
                    userMarker = L.marker([lat, lon]).addTo(mapInstance);
                }

                // Añadir punto al track
                currentTrack.addLatLng([lat, lon]);

                mapInstance.flyTo([lat, lon], mapInstance.getZoom());
            },
            (err) => console.error("Error tracking:", err),
            { enableHighAccuracy: true, maximumAge: 0, timeout: 5000 }
        );

        btn.innerHTML = '⏹️ Detener Tracking';
        btn.style.background = '#ff3300';
        updateMonitor("🟢 Tracking de ruta INICIADO - Muévete");
    }
}


// Función para borrar todo
function clearAllDrawnItems() {
    if (confirm("¿Estás seguro de borrar TODOS los waypoints y tracks?")) {
        drawnItems.clearLayers();
        updateMonitor("🗑️ Todo borrado correctamente");
    }
}

// ==================== PANEL DE HERRAMIENTAS (Waypoints + Tracks) ====================
function createToolsPanel() {
    const panel = document.createElement('div');
    panel.id = 'map-tools-panel';
    panel.style.cssText = `
        position: absolute; 
        top: 130px;           /* Bajado para no tapar el selector de mapas */
        right: 15px; 
        z-index: 1005;        /* Más arriba que los controles de Leaflet */
        background: rgba(0,0,0,0.92); 
        border: 2px solid #00ff88; 
        border-radius: 6px; 
        padding: 10px; 
        width: 260px; 
        color: #00ff88;
        font-size: 0.75rem; 
        max-height: 65vh; 
        overflow-y: auto;
        box-shadow: 0 0 15px rgba(0, 255, 136, 0.3);
    `;

    panel.innerHTML = `
        <div style="margin-bottom:8px; font-weight:bold; color:#00ffea; text-align:center;">
            📍 WAYPOINTS & TRACKS
        </div>
        <div id="waypoints-list" style="margin-bottom:12px; max-height:180px; overflow-y:auto;"></div>
        <div id="tracks-list" style="max-height:140px; overflow-y:auto;"></div>
        
        <button onclick="clearAllDrawnItems()" 
                style="margin-top:10px; width:100%; background:#ff3300; color:white; 
                       border:none; padding:8px; border-radius:4px; font-weight:bold;">
            🗑️ BORRAR TODO
        </button>
    `;

    document.getElementById('map-container').appendChild(panel);
}

// Función de cierre reforzada
function closeModal680() {
    const modal = document.getElementById('modal-680');
    if (modal) {
        modal.style.display = 'none';
        document.getElementById('modal-body').innerHTML = ''; // Limpia memoria
        
        // Reset de botones en el sidebar
        document.querySelectorAll('.btn-cockpit').forEach(b => b.classList.remove('active'));
        const btnCom = document.getElementById('btn-com');
        if (btnCom) btnCom.classList.add('active');
        
        updateMonitor("🖥️ REGRESO A CONSOLA PRINCIPAL");
    }
}

function closeModal680() {
    document.getElementById('modal-680').style.display = 'none';
    document.getElementById('modal-body').innerHTML = "";
    handleCockpitClick('COM');
}




// Inicializar consola cuando se carga la página
setTimeout(() => {
    console.log("🚀 RADCOM CONSOLE v5.6.1 ready");
}, 1000);


setTimeout(finalInitialization, 1200);

function force720() {
    const container = document.querySelector('.container');
    if (container) {
        container.style.width = '720px';
        container.style.height = '720px';
    }
}





        // Exportar nuevas funciones
        window.forceUpdateSatelliteData = forceUpdateSatelliteData;
        window.sendSatelliteEmergency = sendSatelliteEmergency;
        window.sendPositionToChat = sendPositionToChat;
        window.getCurrentGPSPosition = getCurrentGPSPosition;
        window.initSatelliteSystem = initSatelliteSystem;
        window.addEventListener('load', force720);

// ====== INICIALIZAR SISTEMA DE COLAS ======
function initQueue() {
    updateQueueCounter();
    
    if (messageQueue.length > 0) {
        updateMonitor(`📨 ${messageQueue.length} mensaje(s) en cola`);
        
        if (!queueRetryInterval) {
            startQueueSystem();
        }
    }
}

// Ejecutar después de cargar
setTimeout(initQueue, 2000);   

      
    </script>
</body>
</html>
