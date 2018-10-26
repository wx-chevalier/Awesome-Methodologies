# Chrome Extension & Puppeteer CheatSheet

# Puppeteer

## 界面操作

## 脚本执行

```js
const puppeteer = require('puppeteer');

(async () => {
  const browser = await puppeteer.launch();

  const page = await browser.newPage();

  await page.goto('https://example.com'); // Get the "viewport" of the page, as reported by the page.

  const dimensions = await page.evaluate(() => {
    return {
      width: document.documentElement.clientWidth,

      height: document.documentElement.clientHeight,

      deviceScaleFactor: window.devicePixelRatio
    };
  });

  console.log('Dimensions:', dimensions);

  await browser.close();
})();
```

```js
const puppeteer = require('puppeteer');

(async () => {
  const browser = await puppeteer.launch({ headless: false });
  const page = await browser.newPage();
  await page.goto('https://google.com');
  await page.addScriptTag({
    url: 'https://rawgithub.com/marmelab/gremlins.js/master/gremlins.min.js'
  });
  await page.evaluate(() => {
    window.gremlins.createHorde().unleash();
  });
})();
```

## 界面导出

```js
const puppeteer = require('puppeteer');

(async () => {
  const browser = await puppeteer.launch();

  const page = await browser.newPage();

  await page.goto('https://news.ycombinator.com', { waitUntil: 'networkidle' });

  await page.pdf({ path: 'hn.pdf', format: 'A4' });

  await browser.close();
})();
```

## 浏览器操作

- 监听网页请求

```js
const puppeteer = require('puppeteer');

puppeteer.launch().then(async browser => {
  const page = await browser.newPage();
  await page.setRequestInterception(true);
  page.on('request', interceptedRequest => {
    if (
      interceptedRequest.url().endsWith('.png') ||
      interceptedRequest.url().endsWith('.jpg')
    )
      interceptedRequest.abort();
    else interceptedRequest.continue();
  });
  await page.goto('https://example.com');
  await browser.close();
});
```
