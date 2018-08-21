```ts
import { launch } from 'puppeteer';
(async () => {
  const browser = await launch({ headless: false });
  const page = await browser.newPage();
  await page.goto('https://example.com', { waitUntil: 'networkidle' });
  await page.addScriptTag({
    url: 'https://code.jquery.com/jquery-3.2.1.min.js'
  });
  await page.close();
  await browser.close();
})();
```
