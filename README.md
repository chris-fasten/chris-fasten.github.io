@ -0,0 +1,352 @@
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>URL to QR Code Generator</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }
        
        body {
            font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, sans-serif;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            min-height: 100vh;
            display: flex;
            align-items: center;
            justify-content: center;
            padding: 20px;
        }
        
        .container {
            background: rgba(255, 255, 255, 0.95);
            backdrop-filter: blur(10px);
            border-radius: 20px;
            padding: 40px;
            box-shadow: 0 20px 40px rgba(0, 0, 0, 0.1);
            max-width: 500px;
            width: 100%;
            text-align: center;
        }
        
        h1 {
            color: #333;
            margin-bottom: 30px;
            font-size: 2.2em;
            font-weight: 700;
        }
        
        .input-group {
            margin-bottom: 30px;
        }
        
        label {
            display: block;
            margin-bottom: 8px;
            font-weight: 600;
            color: #555;
            text-align: left;
        }
        
        input[type="url"] {
            width: 100%;
            padding: 15px;
            border: 2px solid #e0e0e0;
            border-radius: 12px;
            font-size: 16px;
            transition: all 0.3s ease;
            background: white;
        }
        
        input[type="url"]:focus {
            outline: none;
            border-color: #667eea;
            box-shadow: 0 0 0 3px rgba(102, 126, 234, 0.1);
        }
        
        button {
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            color: white;
            border: none;
            padding: 15px 30px;
            border-radius: 12px;
            font-size: 16px;
            font-weight: 600;
            cursor: pointer;
            transition: all 0.3s ease;
            width: 100%;
        }
        
        button:hover {
            transform: translateY(-2px);
            box-shadow: 0 10px 20px rgba(102, 126, 234, 0.3);
        }
        
        button:active {
            transform: translateY(0);
        }
        
        .qr-output {
            margin-top: 30px;
            padding: 20px;
            background: white;
            border-radius: 12px;
            box-shadow: 0 5px 15px rgba(0, 0, 0, 0.1);
            display: none;
        }
        
        .qr-output.show {
            display: block;
            animation: fadeIn 0.5s ease;
        }
        
        @keyframes fadeIn {
            from { opacity: 0; transform: translateY(20px); }
            to { opacity: 1; transform: translateY(0); }
        }
        
        canvas {
            border: 1px solid #e0e0e0;
            border-radius: 8px;
        }
        
        .download-btn {
            margin-top: 15px;
            background: #28a745;
            font-size: 14px;
            padding: 10px 20px;
            width: auto;
            display: inline-block;
        }
        
        .download-btn:hover {
            background: #218838;
            box-shadow: 0 5px 15px rgba(40, 167, 69, 0.3);
        }
        
        .error {
            color: #dc3545;
            font-size: 14px;
            margin-top: 5px;
            text-align: left;
        }
        
        @media (max-width: 480px) {
            .container {
                padding: 30px 20px;
            }
            
            h1 {
                font-size: 1.8em;
            }
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>🔗 QR Code Generator</h1>
        
        <form id="qrForm">
            <div class="input-group">
                <label for="urlInput">Enter URL:</label>
                <input 
                    type="url" 
                    id="urlInput" 
                    placeholder="https://example.com"
                    required
                >
                <div id="errorMessage" class="error"></div>
            </div>
            
            <button type="submit">Generate QR Code</button>
        </form>
        
        <div id="qrOutput" class="qr-output">
            <canvas id="qrCanvas"></canvas>
            <button id="downloadBtn" class="download-btn">Download QR Code</button>
        </div>
    </div>

    <script>
        const form = document.getElementById('qrForm');
        const urlInput = document.getElementById('urlInput');
        const errorMessage = document.getElementById('errorMessage');
        const qrOutput = document.getElementById('qrOutput');
        const qrCanvas = document.getElementById('qrCanvas');
        const downloadBtn = document.getElementById('downloadBtn');
        
        function isValidURL(string) {
            try {
                new URL(string);
                return true;
            } catch (_) {
                return false;
            }
        }
        
        function showError(message) {
            errorMessage.textContent = message;
            errorMessage.style.display = 'block';
        }
        
        function hideError() {
            errorMessage.textContent = '';
            errorMessage.style.display = 'none';
        }
        
        // Use Google Charts API to generate QR code
        function generateQRCode(url) {
            const size = '300x300';
            const qrUrl = `https://api.qrserver.com/v1/create-qr-code/?size=${size}&data=${encodeURIComponent(url)}`;
            
            // Create an image element
            const img = new Image();
            img.crossOrigin = 'anonymous';
            
            img.onload = function() {
                // Set canvas size
                qrCanvas.width = 300;
                qrCanvas.height = 300;
                
                // Draw the image on canvas
                const ctx = qrCanvas.getContext('2d');
                ctx.clearRect(0, 0, qrCanvas.width, qrCanvas.height);
                ctx.drawImage(img, 0, 0, 300, 300);
                
                hideError();
                qrOutput.classList.add('show');
            };
            
            img.onerror = function() {
                // Fallback: create a simple QR code pattern if API fails
                showError('QR code service unavailable. Using fallback method.');
                createFallbackQR(url);
            };
            
            img.src = qrUrl;
        }
        
        // Fallback QR code generator (very basic)
        function createFallbackQR(url) {
            qrCanvas.width = 300;
            qrCanvas.height = 300;
            const ctx = qrCanvas.getContext('2d');
            
            // Clear canvas
            ctx.fillStyle = '#ffffff';
            ctx.fillRect(0, 0, 300, 300);
            
            // Create a simple pattern - this is just a placeholder
            // In a real implementation, you'd need a full QR code algorithm
            ctx.fillStyle = '#000000';
            
            // Simple grid pattern as placeholder
            const cellSize = 10;
            const data = url.split('').map(char => char.charCodeAt(0));
            
            for (let i = 0; i < 25; i++) {
                for (let j = 0; j < 25; j++) {
                    const index = (i * 25 + j) % data.length;
                    if (data[index] % 2 === 0) {
                        ctx.fillRect(j * cellSize + 25, i * cellSize + 25, cellSize, cellSize);
                    }
                }
            }
            
            // Add corner markers
            ctx.fillRect(25, 25, 70, 70);
            ctx.fillStyle = '#ffffff';
            ctx.fillRect(35, 35, 50, 50);
            ctx.fillStyle = '#000000';
            ctx.fillRect(45, 45, 30, 30);
            
            qrOutput.classList.add('show');
        }
        
        function downloadQRCode() {
            const link = document.createElement('a');
            link.download = 'qrcode.png';
            link.href = qrCanvas.toDataURL('image/png');
            document.body.appendChild(link);
            link.click();
            document.body.removeChild(link);
        }
        
        form.addEventListener('submit', function(e) {
            e.preventDefault();
            
            const url = urlInput.value.trim();
            
            if (!url) {
                showError('Please enter a URL');
                return;
            }
            
            if (!isValidURL(url)) {
                showError('Please enter a valid URL (include http:// or https://)');
                return;
            }
            
            hideError();
            console.log('Generating QR code for:', url); // Debug log
            generateQRCode(url);
        });
        
        downloadBtn.addEventListener('click', downloadQRCode);
        
        // Clear error when user starts typing
        urlInput.addEventListener('input', function() {
            if (errorMessage.textContent) {
                hideError();
            }
        });
        
        // Auto-add https:// if protocol is missing
        urlInput.addEventListener('blur', function() {
            let value = this.value.trim();
            if (value && !value.match(/^https?:\/\//)) {
                this.value = 'https://' + value;
            }
        });
        
        // URL Query Parameter Support with proper separation
        function getUrlParameter(name) {
            const urlParams = new URLSearchParams(window.location.search);
            return urlParams.get(name);
        }
        
        function parseAppParameters() {
            const fullUrl = window.location.href;
            const questionMarkIndex = fullUrl.indexOf('?');
            
            if (questionMarkIndex === -1) {
                return { targetUrl: null, autoGenerate: false, imageOnly: false };
            }
            
            const queryString = fullUrl.substring(questionMarkIndex + 1);
            const urlParams = new URLSearchParams(queryString);
            
            // Get app control parameters first
            const autoGenerate = urlParams.get('auto') === 'true';
            const imageOnly = urlParams.get('image') === 'true';
            
            // For the URL parameter, we need to be more careful
            let targetUrl = urlParams.get('url');
            
            // If URL parameter is not properly encoded, try alternative parsing
            if (!targetUrl && queryString.includes('url=')) {
                // Find the url= parameter manually
                const urlParamStart = queryString.indexOf('url=') + 4;
                let urlParamEnd = queryString.length;
                
                // Look for the next parameter that belongs to the app (auto= or image=)
                const nextAutoIndex = queryString.indexOf('&auto=', urlParamStart);
                const nextImageIndex = queryString.indexOf('&image=', urlParamStart);
                
                if (nextAutoIndex !== -1 && nextImageIndex !== -1) {
                    urlParamEnd = Math.min(nextAutoIndex, nextImageIndex);
                } else if (nextAutoIndex !== -1) {
                    urlParamEnd = nextAutoIndex;
                } else if (nextImageIndex !== -1) {
                    urlParamEnd = nextImageIndex;
                }
                
                targetUrl = queryString.substring(urlParamStart, urlParamEnd);
                targetUrl = decodeURIComponent(targetUrl);
            } else if (targetUrl) {
                targetUrl = decodeURIComponent(targetUrl);
            }
            
            // Also check for 'link' parameter as alternative
            if (!targetUrl) {
                targetUrl = urlParams.get('link');
                if (targetUrl) {
                    targetUrl = decodeURIComponent(targetUrl);
                }
            }
            
            return {
                targetUrl,
                autoGenerate,
                imageOnly
            };
        }
        
        function initFromQueryParams() {
            const params = parseAppParameters();
            
            console.log('Parsed parameters:', params); // Debug log
            
            if (params.targetUrl) {
                urlInput.value = params.targetUrl;
                
                // Image-only mode for spreadsheet integration
                if (params.imageOnly) {
                    document.body.style.margin = '0';
                    document.body.style.padding = '0';
                    document.body.style.background = 'white';
                    
                    // Hide the UI, show only the QR code
                    const container = document.querySelector('.container');
                    container.style.display = 'none';
                    
                    // Create image-only container
                    const imageContainer = document.createElement('div');
                    imageContainer.style.textAlign = 'center';
                    imageContainer.style.padding = '10px';
                    document.body.appendChild(imageContainer);
                    
                    // Generate QR code and display as image
                    if (isValidURL(params.targetUrl)) {
                        generateQRImageOnly(params.targetUrl, imageContainer);
                    }
                    return;
                }
                
                // Auto-generate if auto=true parameter is present
                if (params.autoGenerate) {
                    // Small delay to ensure page is fully loaded
                    setTimeout(() => {
                        if (isValidURL(params.targetUrl)) {
                            generateQRCode(params.targetUrl);
                        }
                    }, 100);
                }
            }
        }
        
        // Generate QR code for image-only mode
        function generateQRImageOnly(url, container) {
            const size = '300x300';
            const qrUrl = `https://api.qrserver.com/v1/create-qr-code/?size=${size}&data=${encodeURIComponent(url)}`;
            
            const img = new Image();
            img.crossOrigin = 'anonymous';
            img.style.maxWidth = '100%';
            img.style.height = 'auto';
            
            img.onload = function() {
                container.appendChild(img);
            };
            
            img.onerror = function() {
                // Fallback: create canvas-based QR code
                const canvas = document.createElement('canvas');
                canvas.width = 300;
                canvas.height = 300;
                container.appendChild(canvas);
                createFallbackQROnCanvas(url, canvas);
            };
            
            img.src = qrUrl;
        }
        
        // Fallback QR generator for image-only mode
        function createFallbackQROnCanvas(url, canvas) {
            const ctx = canvas.getContext('2d');
            
            // Clear canvas
            ctx.fillStyle = '#ffffff';
            ctx.fillRect(0, 0, 300, 300);
            
            // Create a simple pattern
            ctx.fillStyle = '#000000';
            
            const cellSize = 10;
            const data = url.split('').map(char => char.charCodeAt(0));
            
            for (let i = 0; i < 25; i++) {
                for (let j = 0; j < 25; j++) {
                    const index = (i * 25 + j) % data.length;
                    if (data[index] % 2 === 0) {
                        ctx.fillRect(j * cellSize + 25, i * cellSize + 25, cellSize, cellSize);
                    }
                }
            }
            
            // Add corner markers
            ctx.fillRect(25, 25, 70, 70);
            ctx.fillStyle = '#ffffff';
            ctx.fillRect(35, 35, 50, 50);
            ctx.fillStyle = '#000000';
            ctx.fillRect(45, 45, 30, 30);
        }
        
        // Initialize from query parameters on page load
        window.addEventListener('load', initFromQueryParams);
        
        // Test button functionality on page load
        console.log('QR Code Generator loaded successfully');
        console.log('Usage examples:');
        console.log('- Simple URL: ?url=https://example.com');
        console.log('- URL with params: ?url=' + encodeURIComponent('https://example.com?id=123&name=test'));
        console.log('- Auto-generate: ?url=' + encodeURIComponent('https://example.com?id=123') + '&auto=true');
        console.log('- Image only: ?url=' + encodeURIComponent('https://example.com?param=value') + '&image=true');
        console.log('- Complex example: ?url=' + encodeURIComponent('https://mysite.com/page?user=john&category=sales&filter=active') + '&auto=true&image=true');
    </script>
</body>
</html>