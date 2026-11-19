# 狗狗記憶翻牌｜對話紀錄

## 1. 建立遊戲

**使用者**希望用 HTML、CSS、JavaScript 做一款 4 × 4 的記憶翻牌遊戲。16 張牌開局全部蓋上。每次翻兩張，同犬種保留正面並加 1 分，不同犬種則翻回牌背。網頁背景色為 `#E3C59A`，牌背使用 `#A54213` 的花磚花紋。

**助理**提供單一 HTML，使用 CSS Grid 排列卡片、Canvas 繪製八種犬種，再用 JavaScript 洗牌、比對犬種 ID、計分和重新開始。翻錯的牌停留 0.9 秒後蓋回。

八種犬種是伯恩山犬、柴犬、柯基犬、黃金獵犬、拉布拉多犬、法國鬥牛犬、薩摩耶、邊境牧羊犬。

## 2. 加入倒數

**使用者**「在左邊加一個倒數90秒的計時，增加趣味及緊張」

**助理**加入計時器。寬螢幕放在牌桌左側，窄螢幕放在左上方。翻第一張牌才開始，最後 10 秒變紅並跳動。時間到停止翻牌並保留分數。

## 3. 檢查缺少的語法

**使用者**「檢視目前還缺哪些語法」

**助理**檢查已貼上的程式，確認已有計時器介面與開局倒數，還需補三處。

| 位置 | 修正 |
| --- | --- |
| 配滿第 8 組 | 停止倒數並顯示通關 |
| 配對失敗後蓋牌 | 保留時間到的結算訊息 |
| 重新開始 | 停止舊倒數並重設為 90 秒 |

貼到 Markdown 時出現的反斜線與多餘星號，也在下方整合版還原成 HTML、CSS 和 JavaScript 語法。

## 4. 整合與測試

**使用者**「給我全部完整版的，讓我可以直接貼上測試」

**助理**提供完整 HTML。下方附錄以使用者貼出的版本為底，整合三處倒數修正。

1. 複製下方程式碼至 `index.html`，以 Chrome 或 Edge 開啟。
2. 等幾秒，確認仍顯示 90 秒。翻第一張牌才開始倒數。
3. 配對成功要加 1 分，失敗則在約 0.9 秒後蓋牌。配滿 8 組時要停表。
4. 可將 `gameSeconds` 暫時改成 `12`，測試最後 10 秒變紅、歸零後鎖牌和重新開始。測完改回 `90`。

## 附錄｜整合版 HTML

```html
<!DOCTYPE html>

<html lang="zh-Hant">



<head>

    <meta charset="UTF-8">

    <meta name="viewport" content="width=device-width, initial-scale=1.0">

    <title>狗狗記憶翻牌</title>



    <style>

        * {

            box-sizing: border-box;

        }



        body {

            margin: 0;

            min-height: 100vh;

            display: grid;

            place-items: center;

            padding: 24px 16px;

            background-color: #E3C59A;

            color: #472719;

            font-family: system-ui, "Microsoft JhengHei", sans-serif;

        }



        .game {

            width: min(100%, 520px);

        }



        h1 {

            margin: 0 0 6px;

            font-size: clamp(1.65rem, 5vw, 2.4rem);

        }



        .instructions {

            margin: 0 0 18px;

        }



        .toolbar {

            display: flex;

            align-items: center;

            justify-content: space-between;

            gap: 12px;

            margin-bottom: 12px;

        }



        .score {

            margin: 0;

            font-weight: bold;

        }



        button {

            font: inherit;

        }



        .restart {

            padding: 8px 14px;

            border: 0;

            border-radius: 8px;

            background: #472719;

            color: #fff8eb;

            cursor: pointer;

        }



        /* 固定四欄，16 張卡片會排成 4 × 4。 */

        .board {

            display: grid;

            grid-template-columns: repeat(4, minmax(0, 1fr));

            gap: clamp(6px, 2vw, 12px);

        }



        .card {

            position: relative;

            display: grid;

            place-items: center;

            min-width: 0;

            aspect-ratio: 1;

            padding: 0;

            overflow: hidden;

            border: 2px solid #773012;

            border-radius: 12px;

            background-color: #A54213;

            cursor: pointer;

        }



        /* 用重複的四瓣花與菱形組成花磚牌背。 */

        .card::before {

            content: "";

            position: absolute;

            inset: 4px;

            border: 1px solid #f4d4a680;

            border-radius: 8px;

            background-image:

                radial-gradient(ellipse 6px 10px at 50% 30%, #f4d4a6b8 95%, transparent),

                radial-gradient(ellipse 6px 10px at 50% 70%, #f4d4a6b8 95%, transparent),

                radial-gradient(ellipse 10px 6px at 30% 50%, #f4d4a6b8 95%, transparent),

                radial-gradient(ellipse 10px 6px at 70% 50%, #f4d4a6b8 95%, transparent),

                repeating-linear-gradient(45deg, transparent 0 13px, #f4d4a64d 13px 14px),

                repeating-linear-gradient(-45deg, transparent 0 13px, #f4d4a64d 13px 14px);

            background-size: 40px 40px;

            background-position: center;

        }



        .dog-image {

            display: block;

            width: 100%;

            height: 100%;

            visibility: hidden;

        }



        .card.is-open {

            background-color: #fff6e7;

        }



        .card.is-open::before {

            display: none;

        }



        .card.is-open .dog-image {

            visibility: visible;

        }



        .card.is-matched {

            border-color: #517044;

            background-color: #edf1df;

        }



        .card[aria-disabled="true"] {

            cursor: default;

        }



        button:focus-visible {

            outline: 3px solid #472719;

            outline-offset: 3px;

        }



        .message {

            min-height: 1.5em;

            margin: 16px 0 0;

        }



        .play-area {

            position: relative;

        }



        .timer {

            position: absolute;

            top: 0;

            right: calc(100% + 16px);

            width: 108px;

            padding: 10px 8px;

            border: 2px solid #A54213;

            border-radius: 12px;

            background-color: #fff6e7;

            text-align: center;

        }



        .timer-label,

        .timer-value {

            display: block;

        }



        .timer-label {

            font-size: 0.875rem;

        }



        .timer-value {

            font-size: 1.4rem;

            font-variant-numeric: tabular-nums;

        }



        .timer.is-urgent .timer-value {

            color: #a2211b;

            animation: timer-pulse 1s ease-in-out infinite;

        }



        .timer.is-expired .timer-value {

            color: #a2211b;

            animation: none;

        }



        @keyframes timer-pulse {

            50% {

                transform: scale(1.12);

            }

        }



        @media (max-width: 800px) {

            .timer {

                position: static;

                margin-bottom: 12px;

            }

        }



        @media (prefers-reduced-motion: reduce) {

            .timer.is-urgent .timer-value {

                animation: none;

            }

        }

    </style>

</head>



<body>

    <main class="game">

        <h1>狗狗記憶翻牌</h1>

        <p class="instructions">每次翻兩張牌，找到同一種狗狗就得 1 分。</p>



        <div class="toolbar">

            <p class="score">分數 <span id="score">0</span> / 8</p>

            <button class="restart" id="restart" type="button">重新開始</button>

        </div>



        <div class="play-area">

            <div class="timer" id="timer">

                <span class="timer-label">剩餘時間</span>

                <strong class="timer-value"><span id="timeLeft">90</span> 秒</strong>

            </div>



            <div class="board" id="board" role="group" aria-label="記憶翻牌牌桌"></div>

        </div>

        <p class="message" id="message" role="status"></p>

    </main>



    <script>

        // 每個物件代表一種犬種。earShape 決定耳型，其餘欄位控制毛色與斑紋。

        const dogs = [

            { id: "bernese", name: "伯恩山犬", fur: "#332B2A", ears: "#292425", muzzle: "#FFF8E9", earShape: "drop", marking: "blaze", accent: "#BF713F" },

            { id: "shiba", name: "柴犬", fur: "#CA763D", ears: "#AF5830", muzzle: "#FFF0D5", earShape: "point", marking: "cheeks" },

            { id: "corgi", name: "柯基犬", fur: "#D58B40", ears: "#BB6C32", muzzle: "#FFF8E9", earShape: "tall", marking: "blaze" },

            { id: "golden", name: "黃金獵犬", fur: "#DBA552", ears: "#AF7333", muzzle: "#F3C97F", earShape: "long", marking: "ruff" },

            { id: "labrador", name: "拉布拉多犬", fur: "#79503C", ears: "#58382E", muzzle: "#A06D50", earShape: "drop", marking: "broad" },

            { id: "frenchie", name: "法國鬥牛犬", fur: "#B89373", ears: "#806453", muzzle: "#55413C", earShape: "bat", marking: "folds" },

            { id: "samoyed", name: "薩摩耶", fur: "#FFFCF4", ears: "#E4D6C7", muzzle: "#FFFDF9", earShape: "point", marking: "ruff" },

            { id: "collie", name: "邊境牧羊犬", fur: "#38383F", ears: "#292932", muzzle: "#FFFCF1", earShape: "fold", marking: "blaze" }

        ];



        // 耳朵路徑畫在左側，右側以鏡像繪製。

        const earPaths = {

            point: "M28 56 Q17 24 28 16 Q38 17 54 42 Z",

            tall: "M27 57 Q11 15 24 9 Q38 5 54 42 Z",

            drop: "M38 35 Q11 32 13 72 Q17 91 34 75 L49 48 Z",

            long: "M39 35 Q11 33 12 72 Q10 98 27 101 Q43 98 42 72 L50 46 Z",

            bat: "M28 56 Q14 22 20 12 Q34 0 48 37 L54 49 Z",

            fold: "M29 59 Q9 45 17 30 Q27 20 43 31 L52 49 Z"

        };



        const board = document.querySelector("#board");

        const scoreDisplay = document.querySelector("#score");

        const message = document.querySelector("#message");

        const restartButton = document.querySelector("#restart");



        let firstCard = null;

        let score = 0;

        let flipBackTimer = null;

        const timerBox = document.querySelector("#timer");

        const timeDisplay = document.querySelector("#timeLeft");

        const gameSeconds = 90;



        let secondsLeft = gameSeconds;

        let countdownId = null;

        let deadline = null;

        let gameOver = false;



        function stopCountdown() {

            window.clearInterval(countdownId);

            countdownId = null;

        }



        function updateCountdown() {

            // 用實際經過的時間計算，切回分頁時也會更新到正確秒數。

            secondsLeft = Math.max(0, Math.ceil((deadline - Date.now()) / 1000));

            timeDisplay.textContent = secondsLeft;



            timerBox.classList.toggle("is-urgent", secondsLeft > 0 && secondsLeft <= 10);

            timerBox.classList.toggle("is-expired", secondsLeft === 0);



            if (secondsLeft === 0) {

                gameOver = true;

                stopCountdown();

                message.textContent = `時間到！你配對了 ${score} / ${dogs.length} 組。`;

            }

        }



        function startCountdown() {

            deadline = Date.now() + gameSeconds * 1000;

            countdownId = window.setInterval(updateCountdown, 100);

        }

        // 從尾端往前走，把每張牌和包含自己在內的前方隨機一張交換。

        function shuffle(cards) {

            for (let i = cards.length - 1; i > 0; i--) {

                const randomIndex = Math.floor(Math.random() * (i + 1));

                [cards[i], cards[randomIndex]] = [cards[randomIndex], cards[i]];

            }

        }



        function drawDog(dog) {

            const canvas = document.createElement("canvas");

            canvas.className = "dog-image";

            canvas.width = 240;

            canvas.height = 240;

            canvas.setAttribute("aria-hidden", "true");



            const ctx = canvas.getContext("2d");

            ctx.scale(2, 2);



            function ellipse(x, y, width, height, color) {

                ctx.beginPath();

                ctx.ellipse(x, y, width, height, 0, 0, Math.PI * 2);

                ctx.fillStyle = color;

                ctx.fill();

            }



            function fillPath(path, color) {

                ctx.fillStyle = color;

                ctx.fill(new Path2D(path));

            }



            fillPath(earPaths[dog.earShape], dog.ears);

            ctx.save();

            ctx.translate(120, 0);

            ctx.scale(-1, 1);

            fillPath(earPaths[dog.earShape], dog.ears);

            ctx.restore();



            if (dog.marking === "ruff") {

                ellipse(37, 82, 18, 17, dog.fur);

                ellipse(83, 82, 18, 17, dog.fur);

                ellipse(60, 92, 25, 18, dog.fur);

            }



            ellipse(60, 67, dog.marking === "broad" ? 37 : 34, 33, dog.fur);



            if (dog.marking === "blaze") {

                fillPath("M57 35 Q65 41 65 58 L71 77 H49 L55 55 Z", dog.muzzle);

            }



            if (dog.marking === "cheeks") {

                ellipse(39, 78, 13, 10, dog.muzzle);

                ellipse(81, 78, 13, 10, dog.muzzle);

            }



            if (dog.accent) {

                ellipse(44, 56, 8, 4, dog.accent);

                ellipse(76, 56, 8, 4, dog.accent);

                ellipse(37, 80, 8, 9, dog.accent);

                ellipse(83, 80, 8, 9, dog.accent);

            }



            ellipse(60, 81, dog.marking === "broad" ? 24 : 19, 15, dog.muzzle);



            if (dog.marking === "folds") {

                fillPath("M49 52 Q60 44 71 52 Q60 48 49 52 Z", "#806453");

                fillPath("M49 57 Q60 50 71 57 Q60 54 49 57 Z", "#806453");

            }



            ellipse(44, 65, 3.5, 4.5, "#201B1A");

            ellipse(76, 65, 3.5, 4.5, "#201B1A");

            ellipse(45, 63, 1, 1, "#FFFFFF");

            ellipse(77, 63, 1, 1, "#FFFFFF");

            ellipse(60, 77, 7, 4.5, "#241D1B");



            ctx.beginPath();

            ctx.moveTo(53, 86);

            ctx.quadraticCurveTo(60, 93, 67, 86);

            ctx.strokeStyle = "#241D1B";

            ctx.lineWidth = 1.6;

            ctx.lineCap = "round";

            ctx.stroke();



            return canvas;

        }



        function createCard(dog, index) {

            const button = document.createElement("button");

            const position = index + 1;



            button.type = "button";

            button.className = "card";

            button.setAttribute("aria-label", `第 ${position} 張，牌背`);

            button.appendChild(drawDog(dog));



            const card = { dog, button, position };

            button.addEventListener("click", () => flipCard(card));



            return button;

        }



        // 同時更新畫面與卡片標籤，覆蓋時只讀出位置。

        function showCard(card, isOpen) {

            card.button.classList.toggle("is-open", isOpen);

            const description = isOpen ? card.dog.name : "牌背";

            card.button.setAttribute("aria-label", `第 ${card.position} 張，${description}`);

        }



        function flipCard(card) {

            // 等待蓋牌時鎖住操作，也防止重點同一張牌或已配對的牌。

            if (gameOver) {

                return;

            }



            // 玩家點擊時也檢查時間，避免在倒數剛結束時還能翻牌。

            if (deadline !== null) {

                updateCountdown();

            }



            if (gameOver || flipBackTimer !== null || card.button.classList.contains("is-open")) {

                return;

            }



            // 第一張有效的翻牌才開始計時。

            if (deadline === null) {

                startCountdown();

            }



            showCard(card, true);



            if (firstCard === null) {

                firstCard = card;

                message.textContent = `翻到${card.dog.name}，再翻一張。`;

                return;

            }



            const previousCard = firstCard;

            firstCard = null;



            // 只比較犬種 id。同一組牌的 id 相同，配對後累加 1 分。

            if (previousCard.dog.id === card.dog.id) {

                for (const matchedCard of [previousCard, card]) {

                    matchedCard.button.classList.add("is-matched");

                    matchedCard.button.setAttribute("aria-disabled", "true");

                    matchedCard.button.setAttribute(

                        "aria-label",

                        `第 ${matchedCard.position} 張，${matchedCard.dog.name}，已配對`

                    );

                }



                score += 1;

                scoreDisplay.textContent = score;
                if (score === dogs.length) {
                    gameOver = true;
                    stopCountdown();
                    timerBox.classList.remove("is-urgent");
                }



                message.textContent = score === dogs.length

                    ? "通關！你找到了全部 8 組狗狗。"

                    : `${card.dog.name}配對成功，目前 ${score} 分。`;

                return;

            }



            // 配對失敗時讓玩家先看 0.9 秒，接著把這兩張翻回牌背。

            message.textContent = `${previousCard.dog.name}和${card.dog.name}沒有配對。`;



            flipBackTimer = window.setTimeout(() => {

                showCard(previousCard, false);

                showCard(card, false);

                flipBackTimer = null;

                if (!gameOver) {
                    message.textContent = "再選兩張牌。";
                }

            }, 900);

        }



        function startGame() {
            stopCountdown();
            deadline = null;
            gameOver = false;
            secondsLeft = gameSeconds;
            timeDisplay.textContent = secondsLeft;
            timerBox.classList.remove("is-urgent", "is-expired");

            // 重新開始時取消上一局還沒執行的蓋牌計時器。

            window.clearTimeout(flipBackTimer);

            flipBackTimer = null;

            firstCard = null;

            score = 0;

            scoreDisplay.textContent = score;

            message.textContent = "選一張牌開始。";



            // 把八個犬種放入牌組兩次，再洗成 16 張牌。

            const deck = [...dogs, ...dogs];

            shuffle(deck);

            board.replaceChildren(...deck.map(createCard));

        }



        restartButton.addEventListener("click", startGame);

        startGame();

    </script>

</body>



</html>
```
