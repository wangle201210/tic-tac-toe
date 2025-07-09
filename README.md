# tic-tac-toe

超级井字棋

## 规则

双方轮流走棋,默认X先行,〇后行,先完成三子连线者获胜!

屏幕仅显示最新六子位置,其余自动消除,消除规则如下:

落第6子,第1子变暗;

落第7子,第1子消失,第2子变暗;

落第8子,第2子消失,第3子变暗;以此类推。

## 效果图
![](./public/rendering.png)


## Project setup

```bash
pnpm install

# npm install
# yarn install
```

### Compiles and hot-reloads for development

```bash
npm run dev
```

### Compiles and minifies for production

```bash
npm run build
```

## Usage

```bash
git clone https://github.com/wangle201210/tic-tac-toe
cd tic-tac-toe
npm i
npm run dev
```