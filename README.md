# catur.-html
<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Game Catur Simple</title>
    <style>
        * {
            box-sizing: border-box;
            user-select: none;
        }
        body {
            font-family: Arial, sans-serif;
            background-color: #2f3542;
            color: white;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            min-height: 100vh;
            margin: 0;
            padding: 10px;
        }
        h2 {
            margin-bottom: 5px;
        }
        #status {
            font-size: 1.2rem;
            margin-bottom: 15px;
            font-weight: bold;
            color: #eccc68;
        }
        #board {
            display: grid;
            grid-template-columns: repeat(8, 1fr);
            grid-template-rows: repeat(8, 1fr);
            width: 100%;
            max-width: 400px;
            aspect-ratio: 1 / 1;
            border: 4px solid #1e222b;
            box-shadow: 0 10px 25px rgba(0,0,0,0.5);
        }
        .cell {
            display: flex;
            align-items: center;
            justify-content: center;
            font-size: 2rem;
            cursor: pointer;
            position: relative;
        }
        .white-cell { background-color: #f0d9b5; }
        .black-cell { background-color: #b58863; }
        
        .cell.selected {
            background-color: #7a9d54 !important;
        }
        .cell.valid-move::after {
            content: "";
            width: 15px;
            height: 15px;
            background-color: rgba(0, 0, 0, 0.3);
            border-radius: 50%;
            position: absolute;
        }
        .piece {
            width: 100%;
            height: 100%;
            display: flex;
            align-items: center;
            justify-content: center;
            transition: transform 0.1s;
        }
        .piece.white { color: #ffffff; filter: drop-shadow(0px 2px 2px rgba(0,0,0,0.6)); }
        .piece.black { color: #000000; filter: drop-shadow(0px 2px 2px rgba(255,255,255,0.2)); }
        
        #reset-btn {
            margin-top: 20px;
            padding: 10px 20px;
            font-size: 1rem;
            background-color: #ff4757;
            color: white;
            border: none;
            border-radius: 5px;
            cursor: pointer;
            font-weight: bold;
        }
        #reset-btn:active {
            background-color: #ff6b81;
        }
    </style>
</head>
<body>

    <h2>Game Catur</h2>
    <div id="status">Giliran: Putih</div>
    <div id="board"></div>
    <button id="reset-btn" onclick="initGame()">Reset Game</button>

    <script>
        const boardElement = document.getElementById('board');
        const statusElement = document.getElementById('status');
        
        // Simbol Bidak Catur Unicode
        const pieces = {
            'r': '♜', 'n': '♞', 'b': '♝', 'q': '♛', 'k': '♚', 'p': '♟', // Hitam
            'R': '♖', 'N': '♘', 'B': '♗', 'Q': '♕', 'K': '♔', 'P': '♙'  // Putih
        };

        let board = [];
        let turn = 'W'; // 'W' untuk Putih, 'B' untuk Hitam
        let selectedSquare = null;
        let validMoves = [];

        // Posisi awal papan catur
        const initialBoard = [
            ['r', 'n', 'b', 'q', 'k', 'b', 'n', 'r'],
            ['p', 'p', 'p', 'p', 'p', 'p', 'p', 'p'],
            ['', '', '', '', '', '', '', ''],
            ['', '', '', '', '', '', '', ''],
            ['', '', '', '', '', '', '', ''],
            ['', '', '', '', '', '', '', ''],
            ['P', 'P', 'P', 'P', 'P', 'P', 'P', 'P'],
            ['R', 'N', 'B', 'Q', 'K', 'B', 'N', 'R']
        ];

        function initGame() {
            board = JSON.parse(JSON.stringify(initialBoard));
            turn = 'W';
            selectedSquare = null;
            validMoves = [];
            statusElement.innerText = "Giliran: Putih";
            createBoard();
        }

        function createBoard() {
            boardElement.innerHTML = '';
            for (let r = 0; r < 8; r++) {
                for (let c = 0; c < 8; c++) {
                    const cell = document.createElement('div');
                    cell.classList.add('cell');
                    cell.classList.add((r + c) % 2 === 0 ? 'white-cell' : 'black-cell');
                    cell.dataset.row = r;
                    cell.dataset.col = c;

                    const piece = board[r][c];
                    if (piece) {
                        const pieceEl = document.createElement('div');
                        pieceEl.classList.add('piece');
                        pieceEl.classList.add(piece === piece.toUpperCase() ? 'white' : 'black');
                        pieceEl.innerHTML = pieces[piece];
                        cell.appendChild(pieceEl);
                    }

                    // Event handler untuk mobile (touch) dan desktop (click)
                    cell.addEventListener('click', () => handleSquareClick(r, c));
                    boardElement.appendChild(cell);
                }
            }
        }

        function handleSquareClick(r, c) {
            const piece = board[r][c];
            const isWhitePiece = piece && piece === piece.toUpperCase();
            const isBlackPiece = piece && piece === piece.toLowerCase();
            
            // Jika mengklik langkah valid yang sudah ditandai
            if (validMoves.some(m => m.r === r && m.c === c)) {
                movePiece(selectedSquare.r, selectedSquare.c, r, c);
                return;
            }

            // Memilih bidak sesuai giliran
            if ((turn === 'W' && isWhitePiece) || (turn === 'B' && isBlackPiece)) {
                selectedSquare = { r, c };
                validMoves = calculateValidMoves(r, c, piece);
                updateBoardSelection();
            } else {
                // Klik di luar atau bidak musuh tanpa langkah valid = batalkan pilihan
                selectedSquare = null;
                validMoves = [];
                updateBoardSelection();
            }
        }

        function updateBoardSelection() {
            const cells = document.querySelectorAll('.cell');
            cells.forEach(cell => {
                const r = parseInt(cell.dataset.row);
                const c = parseInt(cell.dataset.col);
                
                cell.classList.remove('selected', 'valid-move');
                
                if (selectedSquare && selectedSquare.r === r && selectedSquare.c === c) {
                    cell.classList.add('selected');
                }
                
                if (validMoves.some(m => m.r === r && m.c === c)) {
                    cell.classList.add('valid-move');
                }
            });
        }

        function calculateValidMoves(r, c, piece) {
            let moves = [];
            const type = piece.toLowerCase();
            const isWhite = piece === piece.toUpperCase();

            // Aturan Dasar Pergerakan (Sederhana)
            if (type === 'p') { // Pion
                const dir = isWhite ? -1 : 1;
                // Maju 1 langkah
                if (r + dir >= 0 && r + dir < 8 && board[r + dir][c] === '') {
                    moves.push({ r: r + dir, c });
                    // Maju 2 langkah di awal
                    if ((isWhite && r === 6) || (!isWhite && r === 1)) {
                        if (board[r + 2 * dir][c] === '') moves.push({ r: r + 2 * dir, c });
                    }
                }
                // Makan Serong
                for (let dc of [-1, 1]) {
                    let targetPiece = board[r + dir]?.[c + dc];
                    if (targetPiece && isEnemy(piece, targetPiece)) {
                        moves.push({ r: r + dir, c: c + dc });
                    }
                }
            } 
            else if (type === 'n') { // Kuda
                const kmoves = [
                    [-2,-1], [-2,1], [-1,-2], [-1,2],
                    [1,-2], [1,2], [2,-1], [2,1]
                ];
                for (let m of kmoves) {
                    let nr = r + m[0], nc = c + m[1];
                    if (nr >= 0 && nr < 8 && nc >= 0 && nc < 8) {
                        if (board[nr][nc] === '' || isEnemy(piece, board[nr][nc])) moves.push({ r: nr, c: nc });
                    }
                }
            }
            else if (type === 'r' || type === 'b' || type === 'q' || type === 'k') { // Benteng, Gajah, Ratu, Raja
                let dirs = [];
                if (type === 'r' || type === 'q' || type === 'k') dirs.push([0,1], [0,-1], [1,0], [-1,0]);
                if (type === 'b' || type === 'q' || type === 'k') dirs.push([1,1], [1,-1], [-1,1], [-1,-1]);

                for (let d of dirs) {
                    let nr = r, nc = c;
                    while (true) {
                        nr += d[0]; nc += d[1];
                        if (nr < 0 || nr >= 8 || nc < 0 || nc >= 8) break;
                        
                        if (board[nr][nc] === '') {
                            moves.push({ r: nr, c: nc });
                        } else {
                            if (isEnemy(piece, board[nr][nc])) moves.push({ r: nr, c: nc });
                            break; // Terhalang
                        }
                        if (type === 'k') break; // Raja cuma 1 langkah
                    }
                }
            }
            return moves;
        }

        function isEnemy(p1, p2) {
            if (!p1 || !p2) return false;
            const isP1White = p1 === p1.toUpperCase();
            const isP2White = p2 === p2.toUpperCase();
            return isP1White !== isP2White;
        }

        function movePiece(fromR, fromC, toR, toC) {
            // Eksekusi perpindahan
            board[toR][toC] = board[fromR][fromC];
            board[fromR][fromC] = '';
            
            // Ganti giliran
            turn = (turn === 'W') ? 'B' : 'W';
            statusElement.innerText = `Giliran: ${turn === 'W' ? 'Putih' : 'Hitam'}`;
            
            // Reset pilihan
            selectedSquare = null;
            validMoves = [];
            
            createBoard();
        }

        // Mulai Game Pertama Kali
        initGame();
    </script>
</body>
</html>
