// ================================================================
// SISTEMA DE ID FIJO MEJORADO: APLICA LA CONFIGURACIÓN GUARDADA AL INICIAR
// ================================================================
(function ensureFixedIdOnStartup() {
    console.log("[SISTEMA ID] Verificando configuración de ID fijo...");

    // Esta función se ejecuta justo después de que la página se ha cargado.
    window.addEventListener('load', function() {
        // Pequeña espera para asegurar que todos los elementos del DOM y
        // las funciones de inicialización (como initPeerJSEnhanced) estén listas.
        setTimeout(() => {
            const config = JSON.parse(localStorage.getItem('radcom_config_v4') || '{}');
            
            // Verificamos si la configuración indica que se debe usar un ID fijo.
            if (config.useFixedId === true && config.fixedId) {
                const fixedId = config.fixedId;
                updateMonitor(`🔧 CONFIGURACIÓN DETECTADA: Usar ID Fijo = "${fixedId}"`);
                
                // La función initPeerJSEnhanced ya está diseñada para leer esta configuración
                // y aplicar el ID. Solo necesitamos asegurarnos de que se llame.
                // Si el Peer aún no se ha iniciado, lo iniciamos con el ID fijo.
                if (typeof initPeerJSEnhanced === 'function') {
                    // Es posible que ya se haya llamado a initPeerJSEnhanced una vez.
                    // Si es así, debemos reiniciar el sistema para aplicar el nuevo ID.
                    if (window.peer && typeof window.peer.destroy === 'function') {
                        console.log("[SISTEMA ID] Reiniciando Peer para aplicar ID fijo...");
                        window.peer.destroy();
                        window.peer = null;
                        // Limpiamos las conexiones existentes
                        window.connections = {};
                        window.connectionHealth = {};
                    }
                    
                    // Llamamos a la función de inicialización que leerá la config.
                    // Podemos pasarle un flag para que no espere y use el ID fijo directamente.
                    // Sin embargo, nuestra initPeerJSEnhanced ya lo hace automáticamente.
                    // Simplemente la llamamos.
                    setTimeout(() => {
                         initPeerJSEnhanced();
                         updateMonitor(`✅ ID FIJO APLICADO: ${fixedId.substring(0, 12)}...`);
                    }, 100); // Un pequeño retraso para asegurar la limpieza.
                    
                } else {
                    console.error("[SISTEMA ID] ERROR: La función initPeerJSEnhanced no está disponible.");
                }
            } else {
                console.log("[SISTEMA ID] No se requiere ID fijo según la configuración.");
                // Si no hay ID fijo configurado, pero el sistema ya debería haber iniciado con uno aleatorio,
                // no hacemos nada. El flujo normal de window.onload se encargará.
            }
        }, 500); // Esperamos medio segundo para que todo esté listo.
    });
})();
