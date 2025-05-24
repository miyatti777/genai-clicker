# Design Document: 生成AIクリッカー

## 概要
このリポジトリは、単一の `index.html` から成るブラウザ向けクリッカーゲームです。ユーザーは中央のAIチップをクリックして「デプロイ」を獲得し、建物やアップグレードの購入によって自動生産量(DPS)を増やすことができます。

## 構成
- **HTML/CSS/JavaScript** すべてが `index.html` 内にまとめられています。
- レイアウトはクリック用の中央エリアと、建物・アップグレードを表示する右側パネルに分かれています。
- スタイルはダークテーマにネオングリーンのアクセント色(`"#00ff41"`)を使用しています。

## 主なデータ構造
- `gameData`: 現在のトークン数、総トークン数、クリック回数、所有建物数などゲーム状態を保持。
- `buildingsData`: GPUからAGIインスタンスまで複数の建物定義。各建物は `baseCost` と `baseProduction` を持ち、所有数に応じて生産量が増加します。
- `upgradesData`: 生産量やクリックパワーを向上させるアップグレード定義。
- `achievementsData`: 条件を満たしたときに表示される実績定義。解除すると通知ポップアップが表示されます。
- `newsMessages`: 定期的に画面下部を流れるニュースティッカー用メッセージ。

## 主な機能
- **クリック処理**: `clickAI()` によりトークンを獲得。クリックエフェクトやボタンアニメーションも実装。
- **建物購入・アップグレード**: 所持トークンが一定以上になると購入可能。購入時は `renderBuildings()`・`renderUpgrades()` を通じて表示を更新。
- **自動生産**: `gameLoop()` が100msごとに呼び出され、建物ごとの生産量計算(`calculateTokensPerSecond()`)を行いトークンを増加させます。
- **ゴールデントークン**: 約30秒ごとに出現する可能性があり、クリックするとDPS×77のボーナスを獲得。
- **セーブ／ロード**: `localStorage` にゲームデータを保存し、再読込時にロード可能。
- **モーダルウィンドウ**: 実績一覧・統計情報を表示するモーダル画面を備えています。
- **ニュースティッカー**: 画面下部でランダムなニュースメッセージを流します。
- **キーボードショートカット**: スペースキーでクリック操作が可能。

## 今後の拡張案
- HTML・CSS・JSをファイル分割し、コードの可読性を向上させる。
- さらなる建物やアップグレードの追加。生産量の計算式を柔軟に変更できる仕組みを導入する。
- オンラインランキングや実績共有などのソーシャル要素追加。
- スマートフォン向けUIの改善やレスポンシブデザインの強化。
- サウンドエフェクトやアニメーションの充実。

## 参考箇所
ゲームデータの定義位置例:
```html
        let gameData = {
            tokens: 0,
            totalTokens: 0,
            tokensPerSecond: 0,
            clickPower: 1,
            buildings: {},
            upgrades: new Set(),
            achievements: new Set(),
            totalClicks: 0,
            startTime: Date.now(),
            goldenTokensClicked: 0
        };
```
【F:index.html†L652-L666】

ゴールデントークン生成処理例:
```javascript
        function spawnGoldenToken() {
            const token = document.createElement('div');
            token.className = 'golden-token';
            token.innerHTML = '⭐';
            token.style.left = Math.random() * (window.innerWidth - 60) + 'px';
            token.style.top = Math.random() * (window.innerHeight - 60) + 'px';

            token.onclick = () => {
                const bonus = gameData.tokensPerSecond * 77; // 77秒分のボーナス
                gameData.tokens += bonus;
                gameData.totalTokens += bonus;
                gameData.goldenTokensClicked++;

                showClickEffect(bonus);
                token.remove();
                updateDisplay();
            };
```
【F:index.html†L1119-L1142】

## まとめ
`index.html` にすべてのロジックが集約されたシンプルな構成ですが、クリック、施設管理、アップグレード、実績、ニュースティッカーなど、一般的なクリッカーゲームの要素を網羅しています。今後はコードの分割やUI改善、機能追加などで拡張可能です。
