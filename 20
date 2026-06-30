"""
Web Scraping Demo — a teaching app for Hugging Face Spaces.

Shows the three core steps of scraping visually:
    1. FETCH  — download raw HTML over HTTP
    2. PARSE  — turn HTML into a searchable tree
    3. EXTRACT — pull structured data using CSS selectors
    4. STORE  — export the result as CSV / JSON

Built with Gradio so it runs as-is on a free Hugging Face Space.
"""

import json
import io
import csv
import requests
from bs4 import BeautifulSoup
import gradio as gr

HEADERS = {
    "User-Agent": "Mozilla/5.0 (web-scraping-demo; educational use)"
}

# Practice sites that explicitly allow scraping — safe for a classroom demo.
PRESETS = {
    "Quotes (quotes.toscrape.com)": (
        "https://quotes.toscrape.com/",
        "div.quote span.text",
    ),
    "Books (books.toscrape.com)": (
        "https://books.toscrape.com/",
        "article.product_pod h3 a",
    ),
    "Quotes — authors only": (
        "https://quotes.toscrape.com/",
        "small.author",
    ),
}


def fetch(url):
    """Step 1: download the page. Returns (response, error_message)."""
    try:
        r = requests.get(url, headers=HEADERS, timeout=15)
        r.raise_for_status()
        return r, None
    except requests.RequestException as e:
        return None, f"Could not fetch the page: {e}"


def scrape(url, selector):
    """Run all four steps and return outputs for the UI."""
    if not url:
        return ("Enter a URL first.", "", "", [], None, None)

    # ---- Step 1: FETCH ----
    resp, err = fetch(url)
    if err:
        return (err, "", "", [], None, None)

    status_lines = [
        f"Status code : {resp.status_code}  ({resp.reason})",
        f"Final URL   : {resp.url}",
        f"Content-Type: {resp.headers.get('Content-Type', 'unknown')}",
        f"Page size   : {len(resp.text):,} characters",
        f"Encoding    : {resp.encoding}",
    ]
    fetch_summary = "\n".join(status_lines)

    raw_html = resp.text
    raw_preview = raw_html[:3000] + ("\n\n... (truncated) ..." if len(raw_html) > 3000 else "")

    # ---- Step 2: PARSE ----
    soup = BeautifulSoup(raw_html, "html.parser")
    pretty = soup.prettify()
    parse_preview = pretty[:3000] + ("\n\n... (truncated) ..." if len(pretty) > 3000 else "")

    page_title = soup.title.get_text(strip=True) if soup.title else "(no <title>)"
    n_links = len(soup.find_all("a", href=True))
    n_images = len(soup.find_all("img"))
    n_para = len(soup.find_all("p"))
    parse_summary = (
        f"Page title : {page_title}\n"
        f"Links <a>  : {n_links}\n"
        f"Images<img>: {n_images}\n"
        f"Paragraphs : {n_para}\n\n"
        "BeautifulSoup turned the raw text above into a tree we can now search."
    )

    # ---- Step 3: EXTRACT ----
    selector = (selector or "").strip()
    rows = []
    if selector:
        try:
            elements = soup.select(selector)
        except Exception as e:
            return (fetch_summary, raw_preview, parse_summary + f"\n\nSelector error: {e}", [], None, None)
        for i, el in enumerate(elements, 1):
            text = el.get_text(strip=True)
            href = el.get("href") or el.get("src") or ""
            rows.append([i, text, href])
    extract_table = rows if rows else [["—", "No elements matched this selector.", ""]]

    # ---- Step 4: STORE ----
    csv_path = json_path = None
    if rows:
        csv_path = "scraped_data.csv"
        with open(csv_path, "w", newline="", encoding="utf-8") as f:
            writer = csv.writer(f)
            writer.writerow(["index", "text", "link"])
            writer.writerows(rows)

        json_path = "scraped_data.json"
        records = [{"index": r[0], "text": r[1], "link": r[2]} for r in rows]
        with open(json_path, "w", encoding="utf-8") as f:
            json.dump(records, f, indent=2, ensure_ascii=False)

    return (fetch_summary, raw_preview, parse_summary + "\n\n--- Parsed HTML (pretty) ---\n" + parse_preview,
            extract_table, csv_path, json_path)


def load_preset(name):
    url, sel = PRESETS[name]
    return url, sel


with gr.Blocks(title="Web Scraping — Live Demo") as demo:
    gr.Markdown(
        """
        # Web Scraping in Python — Live Demo
        Enter a URL and a **CSS selector**, then watch the four steps every scraper performs:
        **Fetch → Parse → Extract → Store.**

        Start with a preset (these sites are built for scraping practice), then try changing the selector.
        """
    )

    with gr.Row():
        preset = gr.Dropdown(
            choices=list(PRESETS.keys()),
            value="Quotes (quotes.toscrape.com)",
            label="Preset example",
        )
    with gr.Row():
        url = gr.Textbox(value="https://quotes.toscrape.com/", label="1. URL to scrape")
        selector = gr.Textbox(value="div.quote span.text", label="2. CSS selector (what to extract)")
    run = gr.Button("Scrape it", variant="primary")

    with gr.Tab("1 + 2  Fetch & Parse"):
        fetch_out = gr.Textbox(label="HTTP response (what the server sent back)", lines=6)
        raw_out = gr.Code(label="Raw HTML (a peek at the page source)", language="html")
        parse_out = gr.Textbox(label="After parsing with BeautifulSoup", lines=16)

    with gr.Tab("3  Extract"):
        gr.Markdown("Every row is one element matched by your CSS selector.")
        table_out = gr.Dataframe(headers=["#", "text", "link / src"], label="Extracted data")

    with gr.Tab("4  Store"):
        gr.Markdown("The extracted data, exported the way a real scraper saves it.")
        csv_out = gr.File(label="Download CSV")
        json_out = gr.File(label="Download JSON")

    preset.change(load_preset, inputs=preset, outputs=[url, selector])
    run.click(
        scrape,
        inputs=[url, selector],
        outputs=[fetch_out, raw_out, parse_out, table_out, csv_out, json_out],
    )

    gr.Markdown(
        """
        ---
        **Try this in class:** change the selector to `a.tag` (the topic tags), or switch the preset to
        Books and use `p.price_color` to grab prices. Open your browser's *Inspect* tool on the real
        site to see where these selectors come from.
        """
    )

if __name__ == "__main__":
    demo.launch()
