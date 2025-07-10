<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>ChatGPT Data Viewer</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        body {
            font-family: 'Helvetica Neue', Arial, sans-serif;
            background: #1a1a1a;
            min-height: 100vh;
            padding: 20px;
            color: #d4d4d4;
        }

        .container {
            max-width: 1200px;
            margin: 0 auto;
            background: #252526;
            border-radius: 20px;
            box-shadow: 0 20px 40px rgba(0,0,0,0.3);
            overflow: hidden;
        }

        .header {
            background: #2d2d30;
            padding: 30px;
            text-align: center;
            color: #d4d4d4;
            border-bottom: 1px solid #3e3e42;
        }

        .header h1 {
            font-size: 2.5rem;
            margin-bottom: 10px;
            text-shadow: 0 2px 4px rgba(0,0,0,0.2);
        }

        .upload-area {
            padding: 40px;
            text-align: center;
            border: 3px dashed #4a4a4a;
            margin: 30px;
            border-radius: 15px;
            background: #2d2d30;
            transition: all 0.3s ease;
            color: #d4d4d4;
        }

        .upload-area:hover {
            border-color: #569cd6;
            background: #333335;
        }

        .upload-area.dragover {
            border-color: #569cd6;
            background: #333335;
            transform: scale(1.02);
        }

        .file-input {
            display: none;
        }

        .upload-btn {
            background: #569cd6;
            color: white;
            border: none;
            padding: 15px 30px;
            border-radius: 25px;
            font-size: 1.1rem;
            cursor: pointer;
            transition: all 0.3s ease;
            box-shadow: 0 4px 15px rgba(86, 156, 214, 0.2);
        }

        .upload-btn:hover {
            transform: translateY(-2px);
            box-shadow: 0 6px 20px rgba(86, 156, 214, 0.3);
            background: #4a90c5;
        }

        .controls {
            padding: 20px 30px;
            background: #2d2d30;
            border-bottom: 1px solid #3e3e42;
            display: flex;
            gap: 15px;
            flex-wrap: wrap;
            align-items: center;
        }

        .search-box {
            flex: 1;
            min-width: 250px;
            padding: 12px 20px;
            border: 2px solid #3e3e42;
            border-radius: 25px;
            font-size: 1rem;
            transition: border-color 0.3s ease;
            background: #1e1e1e;
            color: #d4d4d4;
        }

        .search-box:focus {
            outline: none;
            border-color: #569cd6;
        }

        .filter-btn {
            padding: 10px 20px;
            border: 2px solid #569cd6;
            background: #252526;
            color: #569cd6;
            border-radius: 20px;
            cursor: pointer;
            transition: all 0.3s ease;
        }

        .filter-btn.active {
            background: #569cd6;
            color: white;
        }

        .stats {
            background: #1e1e1e;
            padding: 15px 30px;
            border-bottom: 1px solid #3e3e42;
            font-size: 0.9rem;
            color: #9d9d9d;
        }

        .conversations {
            max-height: 70vh;
            overflow-y: auto;
            padding: 20px;
        }

        .conversation {
            margin-bottom: 30px;
            border: 1px solid #3e3e42;
            border-radius: 15px;
            overflow: hidden;
            transition: all 0.3s ease;
            background: #2d2d30;
        }

        .conversation:hover {
            box-shadow: 0 8px 25px rgba(0,0,0,0.3);
            transform: translateY(-2px);
        }

        .conversation-header {
            background: #37373d;
            color: #d4d4d4;
            padding: 20px;
            cursor: pointer;
            display: flex;
            justify-content: space-between;
            align-items: center;
            border-bottom: 1px solid #3e3e42;
        }

        .conversation-title {
            font-size: 1.2rem;
            font-weight: 600;
        }

        .conversation-meta {
            font-size: 0.9rem;
            opacity: 0.9;
        }

        .conversation-content {
            display: none;
            padding: 20px;
            background: #252526;
        }

        .conversation-content.expanded {
            display: block;
        }

        .message {
            margin-bottom: 20px;
            padding: 15px;
            border-radius: 10px;
            animation: fadeIn 0.3s ease;
        }

        @keyframes fadeIn {
            from { opacity: 0; transform: translateY(10px); }
            to { opacity: 1; transform: translateY(0); }
        }

        .message.user {
            background: #4a5568;
            color: #e2e8f0;
            margin-left: 50px;
        }

        .message.assistant {
            background: #2d2d30;
            border-left: 4px solid #4a90c5;
            margin-right: 50px;
            color: #d4d4d4;
        }

        .message-content {
            line-height: 1.6;
            white-space: pre-wrap;
        }

        .message-time {
            font-size: 0.8rem;
            opacity: 0.7;
            margin-top: 8px;
        }

        .no-data {
            text-align: center;
            padding: 60px;
            color: #9d9d9d;
            font-size: 1.2rem;
        }

        .loading {
            text-align: center;
            padding: 40px;
            color: #569cd6;
            font-size: 1.1rem;
        }

        .error {
            background: #2d1f1f;
            border: 1px solid #cd6155;
            color: #e74c3c;
            padding: 15px;
            border-radius: 10px;
            margin: 20px;
        }

        .expand-icon {
            transition: transform 0.3s ease;
        }

        .expanded .expand-icon {
            transform: rotate(180deg);
        }

        @media (max-width: 768px) {
            .header h1 {
                font-size: 2rem;
            }
            
            .message.user {
                margin-left: 20px;
            }
            
            .message.assistant {
                margin-right: 20px;
            }
            
            .controls {
                flex-direction: column;
                align-items: stretch;
            }
            
            .search-box {
                min-width: auto;
            }
        }
    </style>
</head>
<body>
    <div class="container">
        <div class="header">
            <h1>🤖 ChatGPT Data Viewer</h1>
            <p>エクスポートされたChatGPTデータを見やすく表示</p>
        </div>

        <div class="upload-area" id="uploadArea">
            <h3>📁 JSONファイルをアップロード</h3>
            <p>ChatGPTからエクスポートしたJSONファイルをドラッグ&ドロップするか、下のボタンをクリック</p>
            <br>
            <input type="file" id="fileInput" class="file-input" accept=".json">
            <button class="upload-btn" onclick="document.getElementById('fileInput').click()">
                ファイルを選択
            </button>
        </div>

        <div class="controls" id="controls" style="display: none;">
            <input type="text" class="search-box" id="searchBox" placeholder="会話を検索...">
            <button class="filter-btn active" onclick="filterConversations('all', event)">すべて</button>
            <button class="filter-btn" onclick="filterConversations('recent', event)">最近</button>
            <button class="filter-btn" onclick="filterConversations('long', event)">長い会話</button>
        </div>

        <div class="stats" id="stats" style="display: none;"></div>

        <div class="conversations" id="conversations">
            <div class="no-data">
                JSONファイルをアップロードしてください
            </div>
        </div>
    </div>

    <script>
        let conversationsData = [];
        let filteredData = [];
        let currentFilter = 'all';

        // ファイルアップロード処理
        const fileInput = document.getElementById('fileInput');
        const uploadArea = document.getElementById('uploadArea');
        const conversationsDiv = document.getElementById('conversations');
        const controlsDiv = document.getElementById('controls');
        const statsDiv = document.getElementById('stats');
        const searchBox = document.getElementById('searchBox');

        // ドラッグ&ドロップ
        uploadArea.addEventListener('dragover', (e) => {
            e.preventDefault();
            uploadArea.classList.add('dragover');
        });

        uploadArea.addEventListener('dragleave', () => {
            uploadArea.classList.remove('dragover');
        });

        uploadArea.addEventListener('drop', (e) => {
            e.preventDefault();
            uploadArea.classList.remove('dragover');
            const files = e.dataTransfer.files;
            if (files.length > 0) {
                handleFile(files[0]);
            }
        });

        fileInput.addEventListener('change', (e) => {
            if (e.target.files.length > 0) {
                handleFile(e.target.files[0]);
            }
        });

        function handleFile(file) {
            if (!file.name.endsWith('.json')) {
                showError('JSONファイルを選択してください');
                return;
            }

            showLoading();
            
            const reader = new FileReader();
            reader.onload = function(e) {
                try {
                    const data = JSON.parse(e.target.result);
                    processData(data);
                } catch (error) {
                    showError('JSONファイルの読み込みに失敗しました: ' + error.message);
                }
            };
            reader.readAsText(file);
        }

        function processData(data) {
            try {
                conversationsData = data.map(item => ({
                    id: item.id || Math.random().toString(36).substr(2, 9),
                    title: item.title || 'Untitled Conversation',
                    create_time: item.create_time || item.created_at || Date.now(),
                    messages: item.mapping ? extractMessages(item.mapping) : (item.messages || [])
                }));

                filteredData = [...conversationsData];
                displayConversations();
                updateStats();
                
                controlsDiv.style.display = 'flex';
                statsDiv.style.display = 'block';
                uploadArea.style.display = 'none';
            } catch (error) {
                showError('データの処理に失敗しました: ' + error.message);
            }
        }

        function extractMessages(mapping) {
            const messages = [];
            
            for (const [key, value] of Object.entries(mapping)) {
                if (value.message && value.message.content && value.message.content.parts) {
                    const message = value.message;
                    messages.push({
                        role: message.author?.role || 'user',
                        content: message.content.parts.join(''),
                        create_time: message.create_time || Date.now()
                    });
                }
            }
            
            return messages.sort((a, b) => a.create_time - b.create_time);
        }

        function displayConversations() {
            if (filteredData.length === 0) {
                conversationsDiv.innerHTML = '<div class="no-data">条件に合う会話が見つかりません</div>';
                return;
            }

            const html = filteredData.map(conv => `
                <div class="conversation">
                    <div class="conversation-header" onclick="toggleConversation('${conv.id}')">
                        <div>
                            <div class="conversation-title">${escapeHtml(conv.title)}</div>
                            <div class="conversation-meta">
                                ${new Date(conv.create_time * 1000).toLocaleString('ja-JP')} - 
                                ${conv.messages.length} メッセージ
                            </div>
                        </div>
                        <div class="expand-icon">▼</div>
                    </div>
                    <div class="conversation-content" id="content-${conv.id}">
                        ${conv.messages.map(msg => `
                            <div class="message ${msg.role}">
                                <div class="message-content">${escapeHtml(msg.content)}</div>
                                <div class="message-time">
                                    ${new Date(msg.create_time * 1000).toLocaleString('ja-JP')}
                                </div>
                            </div>
                        `).join('')}
                    </div>
                </div>
            `).join('');

            conversationsDiv.innerHTML = html;
        }

        function toggleConversation(id) {
            const content = document.getElementById(`content-${id}`);
            const header = content.previousElementSibling;
            
            if (content.classList.contains('expanded')) {
                content.classList.remove('expanded');
                header.classList.remove('expanded');
            } else {
                content.classList.add('expanded');
                header.classList.add('expanded');
            }
        }

        function filterConversations(type, event) {
            currentFilter = type;
            
            // ボタンの状態を更新
            document.querySelectorAll('.filter-btn').forEach(btn => {
                btn.classList.remove('active');
            });
            if (event && event.target) {
                event.target.classList.add('active');
            }

            const now = Date.now() / 1000;
            const oneWeekAgo = now - (7 * 24 * 60 * 60);

            switch(type) {
                case 'recent':
                    filteredData = conversationsData.filter(conv => conv.create_time > oneWeekAgo);
                    break;
                case 'long':
                    filteredData = conversationsData.filter(conv => conv.messages.length > 10);
                    break;
                default:
                    filteredData = [...conversationsData];
            }

            applySearch();
        }

        function applySearch() {
            const searchTerm = searchBox.value.toLowerCase();
            let data = [...filteredData];
            if (searchTerm) {
                data = data.filter(conv => 
                    conv.title.toLowerCase().includes(searchTerm) ||
                    conv.messages.some(msg => msg.content.toLowerCase().includes(searchTerm))
                );
            }
            filteredData = data;
            displayConversations();
            updateStats();
        }

        searchBox.addEventListener('input', applySearch);

        function updateStats() {
            const totalMessages = filteredData.reduce((sum, conv) => sum + conv.messages.length, 0);
            statsDiv.innerHTML = `
                表示中: ${filteredData.length} 会話 / ${totalMessages} メッセージ 
                (全体: ${conversationsData.length} 会話)
            `;
        }

        function showLoading() {
            conversationsDiv.innerHTML = '<div class="loading">⏳ データを読み込み中...</div>';
        }

        function showError(message) {
            conversationsDiv.innerHTML = `<div class="error">❌ エラー: ${message}</div>`;
        }

        function escapeHtml(text) {
            const div = document.createElement('div');
            div.textContent = text;
            return div.innerHTML;
        }
    </script>
</body>
</html>
