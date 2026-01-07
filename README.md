# Anthropic의 MCP로 Webスクレイピング하기

[![Bright Data Promo](https://github.com/luminati-io/LinkedIn-Scraper/raw/main/Proxies%20and%20scrapers%20GitHub%20bonus%20banner.png)](https://brightdata.co.kr/)

이 가이드는 온디맨드 데이터 추출을 위한 MCP 서버를 설정하고, 개발 도구와 연결하며, Bright Data를 활용해 AI 호환 웹 정보를 즉시 확보하는 방법을 설명합니다.

- [제약 이해하기: LLM이 실제 세계와 상호작용하는 데 왜 도움이 필요한가](#understanding-the-limitation-why-llms-need-help-with-real-world-interaction)
- [MCP의 중요성](#the-importance-of-mcp)
- [Model Context Protocol 이해하기](#understanding-model-context-protocol)
- [MCP 아키텍처 설명](#mcp-architecture-explained)
- [나만의 MCP 서버 개발하기](#developing-your-own-mcp-server)
- [MCP 서버 연결하기](#connecting-your-mcp-server)
- [전문적인 웹 데이터 추출을 위한 Bright Data의 MCP 사용하기](#using-bright-datas-mcp-for-professional-web-data-extraction)
- [추가 읽을거리](#further-reading)

## Understanding the Limitation: Why LLMs Need Help with Real-World Interaction

대규모 언어 모델(LLM)은 방대한 학습 데이터셋을 바탕으로 텍스트를 처리하고 생성하는 데 뛰어납니다. 그러나 중요한 제약이 있습니다. 즉, 외부 세계와 자연스럽게 상호작용할 수 없습니다. 이는 로컬 파일에 접근하거나, 커스텀 스크립트를 실행하거나, 웹사이트에서 최신 정보를 가져오는 기능이 기본적으로 내장되어 있지 않다는 뜻입니다.

기본적인 예를 들어보겠습니다. Claude에게 활성 Amazon 상품 페이지에서 세부 정보를 추출해 달라고 요청하는 것은 추가 도구 없이는 불가능합니다. 왜일까요? 인터넷을 탐색하거나 외부 동작을 트리거할 고유한 기능이 없기 때문입니다.

![claude-without-mcp](https://github.com/luminati-io/web-scraping-with-mcp/blob/main/images/claude-without-mcp.png)

보조 도구 없이 LLM은 실시간 데이터에 의존하거나 외부 시스템과의 통합이 필요한 실무 작업을 수행할 수 없습니다.

바로 여기에서 [Anthropic's Model Context Protocol (MCP)](https://www.anthropic.com/news/model-context-protocol)가 유용합니다. MCP는 LLM이 데이터 추출기, API, 스크립트 같은 외부 도구와 안전하고 표준화된 방식으로 통신할 수 있도록 해줍니다.

다음은 실제로 달라지는 점입니다. 커스텀 MCP 서버를 통합한 뒤, Claude를 통해 구조화된 Amazon 상품 정보를 성공적으로 직접 추출했습니다.

![claude-amazon-product-data-extraction-results](https://github.com/luminati-io/web-scraping-with-mcp/blob/main/images/claude-amazon-product-data-extraction-results.png)

## The Importance of MCP

- **표준화:** MCP는 LLM 기반 시스템이 외부 도구 및 데이터에 연결할 수 있는 통일된 인터페이스를 제공합니다. 이는 API가 웹 통합을 표준화한 방식과 유사합니다. 이를 통해 커스텀 통합의 필요성이 크게 줄어 개발이 가속화됩니다.
- **유연성과 확장성:** 개발자는 도구 통합을 다시 작성하지 않고도 LLM 또는 호스팅 플랫폼을 교체할 수 있습니다. MCP는 `stdio` 같은 여러 통신 방식을 지원하므로 다양한 구성에 적응할 수 있습니다.
- **LLM 기능 강화:** MCP는 LLM을 최신 데이터 및 외부 도구와 연결함으로써 정적인 응답을 넘어설 수 있게 합니다. 이제 LLM은 최신의 관련 정보를 제공하고, 컨텍스트에 따라 실제 세계의 동작을 트리거할 수 있습니다.

> **비유**:
> 
> MCP는 LLM을 위한 USB 인터페이스라고 생각하시면 됩니다. USB가 특별한 드라이버 없이도 다양한 장치(키보드, 프린터, 외장 드라이브)를 호환 기기에 연결할 수 있게 해주듯, MCP는 표준화된 프로토콜을 통해 LLM이 폭넓은 도구에 연결되도록 해줍니다. 매번 커스텀 통합을 할 필요가 없습니다.

## Understanding Model Context Protocol

Model Context Protocol(MCP)은 Anthropic이 개발한 오픈 표준으로, 대규모 언어 모델(LLM)이 외부 도구, API, 데이터 소스와 일관되고 안전한 방식으로 상호작용할 수 있도록 합니다. MCP는 범용 커넥터로 기능하며, LLM이 웹사이트 데이터 추출, 데이터베이스 쿼리, 스크립트 실행과 같은 현실 세계의 작업을 수행할 수 있게 합니다.

Anthropic이 이를 소개했지만 MCP는 개방형이며 확장 가능합니다. 즉, 누구나 표준을 구현하거나 기여할 수 있습니다. [Retrieval-Augmented Generation (RAG)](https://brightdata.co.kr/blog/web-data/rag-explained)를 사용해 본 적이 있다면 이 개념을 이해하기 쉬우실 것입니다. MCP는 경량 JSON-RPC 인터페이스를 통해 상호작용을 표준화함으로써, 모델이 라이브 데이터에 접근하고 동작을 수행할 수 있도록 한다는 점에서 그 아이디어를 확장합니다.

## MCP Architecture Explained

기본적으로 MCP는 AI 모델과 외부 기능 간의 통신을 표준화합니다.

**핵심 아이디어:** 표준화된 인터페이스(일반적으로 `stdio` 같은 전송 계층 위에서 JSON-RPC 2.0 사용)를 통해 LLM(클라이언트를 통해)이 외부 서버가 노출하는 도구를 탐색하고 호출할 수 있습니다.

MCP는 세 가지 핵심 구성 요소를 갖춘 클라이언트-서버 아키텍처로 동작합니다.

1. **MCP Host**: LLM과 외부 도구 간 상호작용을 시작하고 관리하는 환경 또는 애플리케이션입니다. 예로는 _Claude Desktop_ 같은 AI 어시스턴트나 _Cursor_ 같은 IDE가 있습니다.
2. **MCP Client**: 호스트 내부의 구성 요소로, MCP Server와의 연결을 설정하고 유지하며 통신 프로토콜을 처리하고 데이터 교환을 관리합니다.
3. **MCP Server:** (개발자가 만드는) MCP 프로토콜을 구현하고 특정 기능 집합을 노출하는 프로그램입니다. MCP 서버는 데이터베이스, 웹 서비스 또는 (이 글의 경우) 웹사이트(Amazon)와 인터페이스할 수 있습니다. 서버는 다음과 같은 표준화된 방식으로 기능을 노출합니다:
   - **Tools:** 호출 가능한 함수(예: _scrape\_amazon\_product_, _get\_weather\_data_)
   - **Resources:** 정적 데이터를 가져오기 위한 읽기 전용 エンドポイント(예: 파일을 가져오기, JSON 레코드 반환)
   - **Prompts:** 도구 및 리소스와의 LLM 상호작용을 안내하는 사전 정의 템플릿

다음은 MCP 아키텍처 다이어그램입니다:

![mcp-architecture-diagram-host-client-server-connections](https://github.com/luminati-io/web-scraping-with-mcp/blob/main/images/mcp-architecture-diagram-host-client-server-connections.png)

_Image Source: [Model Context Protocol](https://modelcontextprotocol.io/introduction)_

이 구성에서 **host**(Claude Desktop 또는 Cursor IDE)가 **MCP client**를 실행하고, 그 클라이언트가 외부 **MCP server**에 연결합니다. 해당 서버는 tools, resources, prompts를 노출하여 AI가 필요할 때 이를 상호작용할 수 있도록 합니다.

요약하면 워크플로는 다음과 같이 동작합니다:

- 사용자가 _"이 Amazon 링크에서 상품 정보를 가져와."_ 같은 메시지를 보냅니다.
- MCP client가 해당 작업을 처리할 수 있는 등록된 tool이 있는지 확인합니다.
- 클라이언트가 MCP server로 구조화된 리クエスト를 전송합니다.
- MCP server가 적절한 동작을 실행합니다(예: 헤드리스 브라우저 실행).
- 서버가 구조화된 결과를 MCP client로 반환합니다.
- 클라이언트가 결과를 LLM로 전달하고, LLM이 이를 사용자에게 제시합니다.

## Developing Your Own MCP Server

Amazon 상품 페이지에서 데이터를 추출하는 Python MCP server를 구축해 보겠습니다.

![amazon-product-page-example](https://github.com/luminati-io/web-scraping-with-mcp/blob/main/images/amazon-product-page-example.png)

이 서버는 두 가지 tool을 제공합니다. 하나는 HTML을 다운로드하고, 다른 하나는 정리된 정보를 추출합니다. Cursor 또는 Claude Desktop의 LLM client를 통해 서버와 상호작용하게 됩니다.

### Step 1: Preparing Your Environment

먼저 [Python 3](https://www.python.org/downloads/)가 설치되어 있는지 확인합니다. 그런 다음 가상 환경을 생성하고 활성화합니다:

```sh
python -m venv mcp-amazon-scraper
# On macOS/Linux:
source mcp-amazon-scraper/bin/activate
# On Windows:
.\mcp-amazon-scraper\Scripts\activate
```

필요한 라이브러리인 [MCP Python SDK](https://github.com/modelcontextprotocol/python-sdk), [Playwright](https://playwright.dev/python/), [LXML](https://lxml.de/)을 설치합니다.

```sh
pip install mcp playwright lxml
# Install browser binaries for Playwright
python -m playwright install
```

이 명령은 다음을 설치합니다:

- **mcp**: 모든 JSON-RPC 통신 세부 사항을 처리하는 Model Context Protocol 서버/클라이언트를 위한 Python SDK입니다.
- **playwright**: JavaScript 비중이 큰 웹사이트를 렌더링하고 スクレイピング하기 위한 헤드리스 브라우저 기능을 제공하는 브라우저 자동화 라이브러리입니다.
- **lxml**: XPath 쿼리를 사용해 웹 페이지에서 특정 데이터 요소를 쉽게 추출할 수 있게 해주는 빠른 XML/HTML 파싱 라이브러리입니다.

요약하면, MCP Python SDK(`mcp`)가 프로토콜 세부 사항을 모두 처리해 주므로 Claude 또는 Cursor가 자연어 프롬프트로 호출할 수 있는 tool을 노출할 수 있습니다. Playwright는 웹 페이지(包括 JavaScript 콘텐츠)를 완전히 렌더링할 수 있게 해주고, lxml은 강력한 HTML 파싱 기능을 제공합니다.

### Step 2: Starting the MCP Server

`amazon_scraper_mcp.py`라는 Python 파일을 생성합니다. 먼저 필요한 모듈을 임포트하고 `FastMCP` server를 초기화합니다:

```python
import os
import asyncio
from lxml import html as lxml_html
from mcp.server.fastmcp import FastMCP
from playwright.async_api import async_playwright

# Define a temporary file path for the HTML content
HTML_FILE = os.path.join(os.getenv("TMPDIR", "/tmp"), "amazon_product_page.html")

# Initialize the MCP server with a descriptive name
mcp = FastMCP("Amazon Product Scraper")

print("MCP Server Initialized: Amazon Product Scraper")
```

이 코드는 MCP server 인스턴스를 생성합니다. 이제 여기에 tool을 추가하겠습니다.

### Step 3: Implementing the `fetch_page` Tool

이 tool은 URL을 입력으로 받아 Playwright로 페이지에 이동하고, 콘텐츠가 로드될 때까지 기다린 뒤 HTML을 다운로드하여 임시 파일에 저장합니다.

```python
@mcp.tool()
async def fetch_page(url: str) -> str:
    """
    Fetches the HTML content of the given Amazon product URL using Playwright
    and saves it to a temporary file. Returns a status message.
    """
    print(f"Executing fetch_page for URL: {url}")
    try:
        async with async_playwright() as p:
            # Launch headless Chromium browser
            browser = await p.chromium.launch(headless=True)
            page = await browser.new_page()
            # Navigate to the URL with a generous timeout
            await page.goto(url, timeout=90000, wait_until="domcontentloaded")
            # Wait for a key element (e.g., body) to ensure basic loading
            await page.wait_for_selector("body", timeout=30000)
            # Add a small delay for any dynamic content rendering via JavaScript
            await asyncio.sleep(5)

            html_content = await page.content()
            with open(HTML_FILE, "w", encoding="utf-8") as f:
                f.write(html_content)

            await browser.close()
            print(f"Successfully fetched and saved HTML to {HTML_FILE}")
            return f"HTML content for {url} downloaded and saved successfully to {HTML_FILE}."
    except Exception as e:
        error_message = f"Error fetching page {url}: {str(e)}"
        print(error_message)
        return error_message
```

이 비동기 함수는 Amazon 페이지에서 발생할 수 있는 JavaScript 렌더링을 처리하기 위해 Playwright를 사용합니다. `@mcp.tool()` 데코레이터는 이 함수를 server 내에서 호출 가능한 tool로 등록합니다.

### Step 4: Implementing the `extract_info` Tool

이 tool은 `fetch_page`가 저장한 HTML 파일을 읽고, LXML과 XPath 셀렉터로 파싱한 뒤, 추출한 상품 세부 정보를 담은 딕셔너리를 반환합니다.

```python
def _extract_xpath(tree, xpath, default="N/A"):
    """Helper function to extract text using XPath, returning default if not found."""
    try:
        # Use text_content() to get text from node and children, strip whitespace
        result = tree.xpath(xpath)
        if result:
            return result[0].text_content().strip()
        return default
    except Exception:
        return default

def _extract_price(price_str):
    """Helper function to parse price string into a float."""
    if price_str == "N/A":
        return None
    try:
        # Remove currency symbols and commas, handle potential whitespace
        cleaned_price = "".join(filter(str.isdigit or str.__eq__("."), price_str))
        return float(cleaned_price)
    except (ValueError, TypeError):
        return None

@mcp.tool()
def extract_info() -> dict:
    """
    Parses the saved HTML file (downloaded by fetch_page) to extract
    Amazon product details like title, price, rating, features, etc.
    Returns a dictionary of the extracted data.
    """
    print(f"Executing extract_info from file: {HTML_FILE}")
    if not os.path.exists(HTML_FILE):
        return {
            "error": f"HTML file not found at {HTML_FILE}. Please run fetch_page first."
        }

    try:
        with open(HTML_FILE, "r", encoding="utf-8") as f:
            page_html = f.read()

        tree = lxml_html.fromstring(page_html)

        # --- XPath Selectors for Amazon Product Details ---
        title = _extract_xpath(tree, '//span[@id="productTitle"]')
        # Handle different price structures (main price, sale price)
        price_whole = _extract_xpath(tree, '//span[contains(@class, "a-price-whole")]')
        price_fraction = _extract_xpath(
            tree, '//span[contains(@class, "a-price-fraction")]'
        )
        price_str = (
            f"{price_whole}.{price_fraction}"
            if price_whole != "N/A"
            else _extract_xpath(tree, '//span[contains(@class,"a-offscreen")]')
        )  # Fallback to offscreen if needed

        price = _extract_price(price_str)

        # Original price (strike-through)
        original_price_str = _extract_xpath(
            tree, '//span[@class="a-price a-text-price"]//span[@class="a-offscreen"]'
        )
        original_price = _extract_price(original_price_str)

        # Rating
        rating_text = _extract_xpath(tree, '//span[@id="acrPopover"]/@title')
        rating = None
        if rating_text != "N/A":
            try:
                rating = float(rating_text.split()[0])
            except (ValueError, IndexError):
                rating = None

        # Review Count
        reviews_text = _extract_xpath(tree, '//span[@id="acrCustomerReviewText"]')
        review_count = None
        if reviews_text != "N/A":
            try:
                review_count = int(reviews_text.split()[0].replace(",", ""))
            except (ValueError, IndexError):
                review_count = None

        # Availability
        availability = _extract_xpath(
            tree,
            '//div[@id="availability"]//span/text()',
        )

        # Features (bullet points)
        feature_elements = tree.xpath(
            '//div[@id="feature-bullets"]//li//span[@class="a-list-item"]'
        )
        features = [
            elem.text_content().strip()
            for elem in feature_elements
            if elem.text_content().strip()
        ]

        # Calculate Discount
        discount = None
        if price and original_price and original_price > price:
            discount = round(((original_price - price) / original_price) * 100)

        extracted_data = {
            "title": title,
            "price": price,
            "original_price": original_price,
            "discount_percent": discount,
            "rating_stars": rating,
            "review_count": review_count,
            "features": features,
            "availability": availability.strip(),
        }
        print(f"Successfully extracted data: {extracted_data}")
        return extracted_data

    except Exception as e:
        error_message = f"Error parsing HTML: {str(e)}"
        print(error_message)  # Added for logging
        return {"error": error_message}
```

이 함수는 LXML의 `fromstring`으로 HTML을 파싱하고, 견고한 XPath 셀렉터로 원하는 요소를 찾습니다.

### Step 5: Running the Server

마지막으로 `amazon_scraper_mcp.py` 스크립트 끝에 다음 줄을 추가하여 `stdio` 전송 메커니즘으로 server를 시작합니다. 이는 Claude Desktop 또는 Cursor 같은 클라이언트와 통신하는 로컬 MCP server에서 표준으로 사용됩니다.

```python
if __name__ == "__main__":
    print("Starting MCP Server with stdio transport...")
    # Run the server, listening via standard input/output
    mcp.run(transport="stdio")
```

### Complete Source Code

```python
import os
import asyncio
from lxml import html as lxml_html
from mcp.server.fastmcp import FastMCP
from playwright.async_api import async_playwright

# Define a temporary file path for the HTML content
HTML_FILE = os.path.join(os.getenv("TMPDIR", "/tmp"), "amazon_product_page.html")

# Initialize the MCP server with a descriptive name
mcp = FastMCP("Amazon Product Scraper")

print("MCP Server Initialized: Amazon Product Scraper")

@mcp.tool()
async def fetch_page(url: str) -> str:
    """
    Fetches the HTML content of the given Amazon product URL using Playwright
    and saves it to a temporary file. Returns a status message.
    """
    print(f"Executing fetch_page for URL: {url}")
    try:
        async with async_playwright() as p:
            # Launch headless Chromium browser
            browser = await p.chromium.launch(headless=True)
            page = await browser.new_page()
            # Navigate to the URL with a generous timeout
            await page.goto(url, timeout=90000, wait_until="domcontentloaded")
            # Wait for a key element (e.g., body) to ensure basic loading
            await page.wait_for_selector("body", timeout=30000)
            # Add a small delay for any dynamic content rendering via JavaScript
            await asyncio.sleep(5)

            html_content = await page.content()
            with open(HTML_FILE, "w", encoding="utf-8") as f:
                f.write(html_content)

            await browser.close()
            print(f"Successfully fetched and saved HTML to {HTML_FILE}")
            return f"HTML content for {url} downloaded and saved successfully to {HTML_FILE}."
    except Exception as e:
        error_message = f"Error fetching page {url}: {str(e)}"
        print(error_message)
        return error_message

def _extract_xpath(tree, xpath, default="N/A"):
    """Helper function to extract text using XPath, returning default if not found."""
    try:
        # Use text_content() to get text from node and children, strip whitespace
        result = tree.xpath(xpath)
        if result:
            return result[0].text_content().strip()
        return default
    except Exception:
        return default

def _extract_price(price_str):
    """Helper function to parse price string into a float."""
    if price_str == "N/A":
        return None
    try:
        # Remove currency symbols and commas, handle potential whitespace
        cleaned_price = "".join(filter(str.isdigit or str.__eq__("."), price_str))
        return float(cleaned_price)
    except (ValueError, TypeError):
        return None

@mcp.tool()
def extract_info() -> dict:
    """
    Parses the saved HTML file (downloaded by fetch_page) to extract
    Amazon product details like title, price, rating, features, etc.
    Returns a dictionary of the extracted data.
    """
    print(f"Executing extract_info from file: {HTML_FILE}")
    if not os.path.exists(HTML_FILE):
        return {
            "error": f"HTML file not found at {HTML_FILE}. Please run fetch_page first."
        }

    try:
        with open(HTML_FILE, "r", encoding="utf-8") as f:
            page_html = f.read()

        tree = lxml_html.fromstring(page_html)

        # --- XPath Selectors for Amazon Product Details ---
        title = _extract_xpath(tree, '//span[@id="productTitle"]')
        # Handle different price structures (main price, sale price)
        price_whole = _extract_xpath(tree, '//span[contains(@class, "a-price-whole")]')
        price_fraction = _extract_xpath(
            tree, '//span[contains(@class, "a-price-fraction")]'
        )
        price_str = (
            f"{price_whole}.{price_fraction}"
            if price_whole != "N/A"
            else _extract_xpath(tree, '//span[contains(@class,"a-offscreen")]')
        )  # Fallback to offscreen if needed

        price = _extract_price(price_str)

        # Original price (strike-through)
        original_price_str = _extract_xpath(
            tree, '//span[@class="a-price a-text-price"]//span[@class="a-offscreen"]'
        )
        original_price = _extract_price(original_price_str)

        # Rating
        rating_text = _extract_xpath(tree, '//span[@id="acrPopover"]/@title')
        rating = None
        if rating_text != "N/A":
            try:
                rating = float(rating_text.split()[0])
            except (ValueError, IndexError):
                rating = None

        # Review Count
        reviews_text = _extract_xpath(tree, '//span[@id="acrCustomerReviewText"]')
        review_count = None
        if reviews_text != "N/A":
            try:
                review_count = int(reviews_text.split()[0].replace(",", ""))
            except (ValueError, IndexError):
                review_count = None

        # Availability
        availability = _extract_xpath(
            tree,
            '//div[@id="availability"]//span/text()',
        )

        # Features (bullet points)
        feature_elements = tree.xpath(
            '//div[@id="feature-bullets"]//li//span[@class="a-list-item"]'
        )
        features = [
            elem.text_content().strip()
            for elem in feature_elements
            if elem.text_content().strip()
        ]

        # Calculate Discount
        discount = None
        if price and original_price and original_price > price:
            discount = round(((original_price - price) / original_price) * 100)

        extracted_data = {
            "title": title,
            "price": price,
            "original_price": original_price,
            "discount_percent": discount,
            "rating_stars": rating,
            "review_count": review_count,
            "features": features,
            "availability": availability.strip(),
        }
        print(f"Successfully extracted data: {extracted_data}")
        return extracted_data

    except Exception as e:
        error_message = f"Error parsing HTML: {str(e)}"
        print(error_message)  # Added for logging
        return {"error": error_message}

if __name__ == "__main__":
    print("Starting MCP Server with stdio transport...")
    # Run the server, listening via standard input/output
    mcp.run(transport="stdio")
```

## Connecting Your MCP Server

이제 server script가 준비되었으니, Claude Desktop 및 Cursor 같은 MCP client에 연결해 보겠습니다.

### Setting Up with Claude Desktop

**Step 1:** Claude Desktop을 엽니다.

**Step 2:** `Settings` -> `Developer` -> `Edit Config`로 이동합니다. 그러면 기본 텍스트 편집기에서 `claude_desktop_config.json` 파일이 열립니다.

![claude-desktop-settings-menu-navigation](https://github.com/luminati-io/web-scraping-with-mcp/blob/main/images/claude-desktop-settings-menu-navigation.png)

**Step 3:** `mcpServers` 키 아래에 server 항목을 추가합니다. `args`의 경로를 `amazon_scraper_mcp.py` 파일의 절대 경로로 바꾸는 것을 잊지 마십시오.

```json
{
  "mcpServers": {
    "amazon_product_scraper": {
      "command": "python",  // Or python3 if needed
      "args": ["/full/path/to/your/amazon_scraper_mcp.py"], // <-- IMPORTANT: Use the correct absolute path
    }
  }
}
```

**Step 4:** `claude_desktop_config.json` 파일을 저장한 뒤, 변경 사항이 적용되도록 Claude Desktop을 완전히 종료했다가 다시 실행합니다.

**Step 5:** 이제 Claude Desktop의 채팅 입력 영역에 작은 도구 아이콘(망치 🔨 같은)이 표시되어야 합니다.

![claude-desktop-mcp-tools-icon-interface](https://github.com/luminati-io/web-scraping-with-mcp/blob/main/images/claude-desktop-mcp-tools-icon-interface.png)

**Step 6:** 이를 클릭하면 `fetch_page` 및 `extract_info` tools를 포함한 "Amazon Product Scraper"가 나열되어야 합니다.

![claude-available-mcp-tools-dialog-amazon-scraper](https://github.com/luminati-io/web-scraping-with-mcp/blob/main/images/claude-available-mcp-tools-dialog-amazon-scraper.png)

**Step 7:** 예를 들어 다음과 같은 프롬프트를 보냅니다: _"Get the current price, original price, and rating for this Amazon product: [https://www.amazon.com/dp/B09C13PZX7](https://www.amazon.com/dp/B09C13PZX7)"._

**Step 8:** Claude는 외부 도구가 필요함을 감지하고 먼저 `fetch_page`, 그 다음 `extract_info`를 실행할 권한을 요청합니다. 각 tool에 대해 "Allow for this chat"를 클릭합니다.

![mcp-permission-dialog-fetch-page-amazon-tool](https://github.com/luminati-io/web-scraping-with-mcp/blob/main/images/mcp-permission-dialog-fetch-page-amazon-tool.png)

**Step 9:** 권한을 부여하면 MCP server가 tools를 실행합니다. Claude는 구조화된 데이터를 수신한 후 채팅에서 이를 제시합니다.

![claude-amazon-product-data-extraction-results](https://github.com/luminati-io/web-scraping-with-mcp/blob/main/images/claude-amazon-product-data-extraction-results-2.png)

### Setting Up with Cursor

Cursor(AI-first IDE)의 과정도 유사합니다.

**Step 1:** Cursor를 엽니다.

**Step 2:** `Settings` ⚙️로 이동한 뒤 `MCP` 섹션으로 이동합니다.

![cursor-ide-add-new-global-mcp-server-settings](https://github.com/luminati-io/web-scraping-with-mcp/blob/main/images/cursor-ide-add-new-global-mcp-server-settings.png)

**Step 3:** "+Add a new global MCP Server"를 클릭합니다. 그러면 `mcp.json` 구성 파일이 열립니다. server 항목을 추가하되, 스크립트의 **절대 경로**를 사용합니다.

![cursor-mcp-json-configuration-file-amazon-scraper](https://github.com/luminati-io/web-scraping-with-mcp/blob/main/images/cursor-mcp-json-configuration-file-amazon-scraper.png)

**Step 4:** `mcp.json` 파일을 저장하면 "amazon\_product\_scraper"가 목록에 표시되고, 실행 및 연결 상태를 나타내는 녹색 점이 표시되기를 기대할 수 있습니다.

![cursor-ide-configured-amazon-scraper-mcp-settings](https://github.com/luminati-io/web-scraping-with-mcp/blob/main/images/cursor-ide-configured-amazon-scraper-mcp-settings.png)

**Step 5:** Cursor의 채팅 기능(`Cmd+l` 또는 `Ctrl+l`)을 사용합니다.

**Step 6:** 예를 들어 다음과 같은 프롬프트를 보냅니다: "_Extract all available product data from this Amazon URL: [https://www.amazon.com/dp/B09C13PZX7](https://www.amazon.com/dp/B09C13PZX7). Format the output as a structured JSON object"_.

**Step 7:** Claude Desktop과 마찬가지로 Cursor는 `fetch_page` 및 `extract_info` tools 실행 권한을 요청합니다. 요청을 승인합니다("Run Tool").

**Step 8:** Cursor는 상호작용 흐름을 표시하며, MCP tool 호출을 보여준 다음 `extract_info` tool이 반환한 구조화된 JSON 데이터를 최종적으로 제시합니다.

![cursor-ide-amazon-product-data-extraction-json-results](https://github.com/luminati-io/web-scraping-with-mcp/blob/main/images/cursor-ide-amazon-product-data-extraction-json-results.png)
다음은 Cursor에서의 JSON 출력 예시입니다:

```json
{
  "title": "Razer Basilisk V3 Customizable Ergonomic Gaming Mouse: Fastest Gaming Mouse Switch - Chroma RGB Lighting - 26K DPI Optical Sensor - 11 Programmable Buttons - HyperScroll Tilt Wheel - Classic Black",
  "price": 39.99,
  "original_price": 69.99,
  "discount_percent": 43,
  "rating_stars": 4.6,
  "review_count": 7782,
  "features": [
    "ICONIC ERGONOMIC DESIGN WITH THUMB REST — PC gaming mouse favored by millions worldwide with a form factor that perfectly supports the hand while its buttons are optimally positioned for quick and easy access",
    "11 PROGRAMMABLE BUTTONS — Assign macros and secondary functions across 11 programmable buttons to execute essential actions like push-to-talk, ping, and more",
    "HYPERSCROLL TILT WHEEL — Speed through content with a scroll wheel that free-spins until its stopped or switch to tactile mode for more precision and satisfying feedback that's ideal for cycling through weapons or skills",
    "11 RAZER CHROMA RGB LIGHTING ZONES — Customize each zone from over 16.8 million colors and countless lighting effects, all while it reacts dynamically with over 150 Chroma integrated games",
    "OPTICAL MOUSE SWITCHES GEN 2 — With zero unintended misclicks these switches provide crisp, responsive execution at a blistering 0.2ms actuation speed for up to 70 million clicks",
    "FOCUS+ 26K DPI OPTICAL SENSOR — Best-in-class mouse sensor with intelligent functions flawlessly tracks movement with zero smoothing, allowing for crisp response and pixel-precise accuracy",
    // ... (other features)
  ],
  "availability": "In Stock"
}
```

이는 MCP의 유연성을 보여줍니다. 동일한 server가 서로 다른 client 애플리케이션과도 매끄럽게 동작합니다.

## Using Bright Data's MCP for Professional Web Data Extraction

Bright Data의 엔터프라이즈급 [Model Context Protocol (MCP)](https://github.com/luminati-io/brightdata-mcp) 솔루션은 자체 관리형 MCP server의 복잡성(예: プロキシ 관리, [アンチボット 내비게이션](https://brightdata.co.kr/blog/web-data/anti-scraping-techniques), 스케일링 과제)을 제거하고, [AI agents](https://brightdata.co.kr/use-cases/apps-agents) 및 LLM과의 원활한 통합을 제공합니다.

Bright Data의 MCP에 연결하면 SERP 결과 및 접근이 어려운 사이트를 포함한 공개 웹 데이터에 즉시 접근할 수 있으며, AI 워크플로에 최적화되어 있습니다.

MCP는 [Web Unlocker](https://brightdata.co.kr/products/web-unlocker), [SERP API](https://brightdata.co.kr/products/serp-api), [Web Scraper API](https://brightdata.co.kr/products/web-scraper), [Scraping Browser](https://brightdata.co.kr/products/scraping-browser) 같은 도구를 통해 강력한 웹 추출 프레임워크를 제공합니다:

- **[AI-Ready Data](https://brightdata.co.kr/use-cases/data-for-ai):** 사전 구조화된 콘텐츠로, 전처리가 필요 없습니다.
- **확장성 및 신뢰성:** 지연 없이 대용량을 지원합니다.
- **차단 및 CAPTCHA 우회:** 고급 アンチボット 기능을 제공합니다.
- **글로벌 IP 커버리지:** [Bright Data proxies](https://brightdata.co.kr/proxy-types)를 통해 195개 국가에서 접근합니다.
- **원활한 통합:** 어떤 MCP client에서도 빠르게 설정할 수 있습니다.

### Prerequisites for Bright Data MCP

Bright Data MCP 통합을 시작하기 전에 다음을 확인하십시오:

1. **Bright Data Account:** [brightdata.com](https://brightdata.co.kr/)에서 등록합니다. 첫 사용자는 테스트용 무료 크레딧을 받습니다.
2. **API Token:** Bright Data 계정 설정에서 API token을 확보합니다([User Settings Page](https://brightdata.co.kr/cp/setting/users)).
3. **Web Unlocker Zone:** Bright Data 제어 패널에서 [Web Unlocker proxy](https://docs.brightdata.com/scraping-automation/web-unlocker/quickstart) zone을 생성합니다. `mcp_unlocker`처럼 기억하기 쉬운 식별자를 선택하십시오(필요 시 환경 변수로 나중에 변경 가능).
4. **(Optional) Scraping Browser Zone:** 고급 브라우저 자동화 기능(예: 복잡한 JavaScript 상호작용 또는 스크린샷)이 필요하다면 [Scraping Browser zone](https://docs.brightdata.com/scraping-automation/scraping-browser/quickstart)을 생성하십시오. 이 zone에 대해 제공되는 인증 정보(Username 및 Password)를(**Overview** 탭에서) 기록하십시오. 일반적으로 `brd-customer-ACCOUNT_ID-zone-ZONE_NAME:PASSWORD` 형식입니다.

### Quickstart: Configuring Bright Data MCP for Claude Desktop

**Step 1:** Bright Data MCP server는 일반적으로 Node.js에 번들로 포함된 `npx`로 실행됩니다. 필요하다면 [공식 웹사이트](https://nodejs.org/en/download)에서 Node.js를 설치하십시오.

**Step 2:** Claude Desktop -> `Settings` -> `Developer` -> `Edit Config` (`claude_desktop_config.json`)를 엽니다.

**Step 3:** `mcpServers` 아래에 Bright Data server 구성을 삽입합니다. 플레이스홀더를 실제 자격 증명으로 바꾸십시오.

```json
{
  "mcpServers": {
    "Bright Data": { // Choose a name for the server
      "command": "npx",
      "args": ["@brightdata/mcp"],
      "env": {
        "API_TOKEN": "YOUR_BRIGHTDATA_API_TOKEN", // Paste your API token here
        "WEB_UNLOCKER_ZONE": "mcp_unlocker",     // Your Web Unlocker zone name
        // Optional: Add if using Scraping Browser tools
        "BROWSER_AUTH": "brd-customer-ACCOUNTID-zone-YOURZONE:PASSWORD"
      }
    }
  }
}
```

**Step 4:** 구성 파일을 저장한 뒤 Claude Desktop을 재시작합니다.

**Step 5:** Claude Desktop에서 망치 아이콘(🔨)에 마우스를 올리면, 이제 여러 MCP tool을 사용할 수 있어야 합니다.

![claude-desktop-interface-with-mcp-tools-available](https://github.com/luminati-io/web-scraping-with-mcp/blob/main/images/claude-desktop-interface-with-mcp-tools-available.png)

スクレイ퍼를 제한할 가능성이 있는 웹사이트로 알려진 Zillow에서 데이터 추출을 시도해 보겠습니다. Claude에 다음과 같이 프롬프트를 입력하십시오: "_Extract key property data in JSON format from this Zillow URL: [https://www.zillow.com/apartments/arverne-ny/the-tides-at-arverne-by-the-sea/ChWHPZ/](https://www.zillow.com/apartments/arverne-ny/the-tides-at-arverne-by-the-sea/ChWHPZ/)_"

![bright-data-mcp-zillow-property-extraction-process](https://github.com/luminati-io/web-scraping-with-mcp/blob/main/images/bright-data-mcp-zillow-property-extraction-process.png)

Claude가 필요한 Bright Data MCP tools를 사용하도록 허용하십시오. Bright Data의 MCP server가 기반 복잡성(プロキシ 로테이션, 필요 시 Scraping Browser를 통한 JavaScript 렌더링)을 처리합니다.

Bright Data server가 추출을 수행하고 구조화된 데이터를 전달하면, Claude가 이를 표시합니다.

![zillow-property-data-json-structure-bright-data-mcp](https://github.com/luminati-io/web-scraping-with-mcp/blob/main/images/zillow-property-data-json-structure-bright-data-mcp.png)

가능한 출력의 예시는 다음과 같습니다:

```json
{
  "propertyInfo": {
    "name": "The Tides At Arverne By The Sea",
    "address": "190 Beach 69th St, Arverne, NY 11692",
    "propertyType": "Apartment building",
    // ... more info
  },
  "rentPrices": {
    "studio": { "startingPrice": "$2,750", /* ... */ },
    "oneBed": { "startingPrice": "$2,900", /* ... */ },
    "twoBed": { "startingPrice": "$3,350", /* ... */ }
  },
  // ... amenities, policies, etc.
}
```

**또 다른 예: Hacker News 헤드라인**

더 간단한 쿼리로는 다음이 있습니다: "_Give me the titles of the latest 5 news articles from Hacker News_".

![hacker-news-latest-articles-mcp-extraction-results](https://github.com/luminati-io/web-scraping-with-mcp/blob/main/images/hacker-news-latest-articles-mcp-extraction-results.png)

이는 Bright Data의 MCP server가 AI 워크플로 내에서 동적이거나 강하게 보안된 웹 콘텐츠에도 직접 접근하는 과정을 얼마나 단순화하는지 보여줍니다.

## Further Reading

AI 및 대규모 언어 모델(LLM)에 대해 더 깊이 있는 지식을 얻기 위해, 이전 가이드 중 일부를 선별하여 소개합니다:

- [Top Sources for Finding LLM Training Data](https://brightdata.co.kr/blog/web-data/llm-training-data)
- [Web Scraping with LLaMA 3: Turn Any Website into Structured JSON](https://brightdata.co.kr/blog/web-data/web-scraping-with-llama-3)
- [Web Scraping With LangChain and Bright Data](https://brightdata.co.kr/blog/web-data/web-scraping-with-langchain-and-bright-data)
- [How To Create a RAG Chatbot With GPT-4o Using SERP Data](https://brightdata.co.kr/blog/web-data/build-a-rag-chatbot)

## Conclusion

Anthropic의 Model Context Protocol은 AI 시스템이 외부 세계와 상호작용하는 방식에 근본적인 변화를 가져옵니다. 특정 작업을 위해 커스텀 MCP server를 구축할 수 있습니다. Bright Data의 MCP 통합은 アンチボット 보호를 회피하고 [AI-ready 구조화 데이터](https://brightdata.co.kr/use-cases/data-for-ai)를 제공하는 엔터프라이즈급 Webスクレイピング 기능을 전달함으로써 이를 한층 더 강화합니다.

지금 [AI solutions](https://brightdata.co.kr/ai)에 등록하고 무료로 사용해 보십시오!