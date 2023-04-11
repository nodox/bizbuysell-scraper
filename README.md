#  How to scrape BizBuySell with Python

Hey friends! Back again with another python web scraping project.

A founder reached out to me to know if it was possible to scrape BizBuySell for businesses for sale in a specific state. They needed the data for business development.

At first I ran into issues with bot blocking. However, with a bit more research, I managed to get the data. The site was not too difficult compared to others. No dynamic javascript which means no need to use selenium. I only fetched the first 12 pages to keep the demo short.

## Code Snippet
```python

from pythonjsonlogger import jsonlogger
from aiolimiter import AsyncLimiter
from urllib.parse import urlparse
import asyncio
import aiohttp
import logging
import time
from pprint import pprint as pp
import random
import aiofiles

logger = logging.getLogger(__name__)
logger.setLevel(logging.INFO)
logHandler = logging.StreamHandler()
formatter = jsonlogger.JsonFormatter()
logHandler.setFormatter(formatter)
logger.addHandler(logHandler)

async def HTTPClientDownloader(url, settings):
    host = urlparse(url).hostname
    max_tcp_connections = settings['max_tcp_connections']

    async with settings['rate_per_host'][host]["limit"]:
        connector = aiohttp.TCPConnector(limit=max_tcp_connections)

        async with aiohttp.ClientSession(connector=connector) as session:
            start_time = time.perf_counter()  # Start timer
            safari_agents = [
                'Safari/17612.3.14.1.6 CFNetwork/1327.0.4 Darwin/21.2.0',  # works!
            ]
            user_agent = random.choice(safari_agents)

            headers = {
                'User-Agent': user_agent
            }

            proxy = None
            html = None
            async with session.get(url, proxy=proxy, headers=headers) as response:
                html = await response.text()
                end_time = time.perf_counter()  # Stop timer
                elapsed_time = end_time - start_time  # Calculate time taken to get response
                status = response.status

                logger.info(
                    msg=f"status={status}, url={url}",
                    extra={
                        "elapsed_time": f"{elapsed_time:4f}",
                    }
                )

                dir = "./data"
                idx = url.split(
                    "https://www.bizbuysell.com/new-york-businesses-for-sale/")[-1]
                loc = f"{dir}/bizbuysell-ny-{idx}.html"

                async with aiofiles.open(loc, mode="w") as fd:
                    await fd.write(html)


async def dispatch(url, settings):
    await HTTPClientDownloader(url, settings)

async def main(start_urls, settings):
    tasks = []
    for url in start_urls:
        task = asyncio.create_task(dispatch(url, settings))
        tasks.append(task)

    results = await asyncio.gather(*tasks)
    print(f"total requests", len(results))

if __name__ == '__main__':
    settings = {
        "max_tcp_connections": 1,
        "proxies": [
            "http://localhost:8765",
        ],
        "rate_per_host": {
            'www.bizbuysell.com': {
                "limit": AsyncLimiter(10, 60), # 10 reqs/min
            },
        }
    }

    start_urls = []
    start, end = 1, 13  # demo purpose
    for i in range(start, end):
        url = f"https://www.bizbuysell.com/new-york-businesses-for-sale/{i}"
        start_urls.append(url)

    asyncio.run(main(start_urls, settings))

```

# About Me

I'm Steven, an software engineer in love with all things web scraping and distributed systems. I worked at Twitter as an SRE migrating 500,000 bare metal servers from Aurora Mesos to Kubernetes.

On Nov 3, 2022 I was laid off. Since then I've spent the time doing deep dives and writing about my journey.

# Web Scraping Course

I'm creating a web scraping course to so you don't have to spend months learning how to:

- analyze a website to determine the best way to scrape data
- use proxies to scrape without getting blocked by Cloudflare, Datadome, or PerimeterX
- scrape web sites with Javascript
- build your own web scraping framework
- build scalable infrastructure for scraping

[Join the prelaunch to gain free access before it becomes a paid course.](https://stevennatera.gumroad.com/l/isfsd)
