# Nature Metadata Export SOP

Target page: <https://www.nature.com/nature/research-articles>

1. Review [the repository disclaimer](../DISCLAIMER.md), Springer Nature's
   current text-and-data-mining guidance, and the publisher's current terms.
2. Open the target page.
3. Open the browser Web Inspector and select the Console.
4. Set `maxPages` to a small number, then paste and run the script once.

```javascript
(async () => {
    const maxPages = 5;
    const rows = [];

    for (let page = 1; page <= maxPages; page++) {
        const pageUrl = page === 1
            ? "https://www.nature.com/nature/research-articles"
            : `https://www.nature.com/nature/research-articles?page=${page}`;
        const html = await fetch(pageUrl).then(response => response.text());
        const doc = new DOMParser().parseFromString(html, "text/html");

        doc.querySelectorAll("article").forEach(article => {
            const link = article.querySelector("a[href*='/articles/']");
            if (!link) return;
            const dateNode = article.querySelector("time, [datetime]");
            rows.push({
                page,
                title: link.innerText.trim(),
                date: dateNode ? dateNode.innerText.trim() : "",
                url: new URL(link.getAttribute("href"), location.origin).href,
                text: article.innerText.trim().replace(/\s+/g, " ")
            });
        });
    }

    const seen = new Set();
    const unique = rows.filter(row => {
        if (seen.has(row.url)) return false;
        seen.add(row.url);
        return true;
    });

    const escapeCSV = value =>
        String(value ?? "").replaceAll('"', '""');
    const csv = [
        "page,title,date,url,text",
        ...unique.map(row =>
            `"${escapeCSV(row.page)}","${escapeCSV(row.title)}","${escapeCSV(row.date)}","${escapeCSV(row.url)}","${escapeCSV(row.text)}"`
        )
    ].join("\n");

    const timestamp = new Date().toISOString().slice(0, 16)
        .replace("T", "_")
        .replace(":", "-");
    const blob = new Blob([csv], { type: "text/csv;charset=utf-8" });
    const downloadUrl = URL.createObjectURL(blob);
    const anchor = document.createElement("a");
    anchor.href = downloadUrl;
    anchor.download = `nature_${timestamp}_pages_1-${maxPages}.csv`;
    document.body.appendChild(anchor);
    anchor.click();
    anchor.remove();
    URL.revokeObjectURL(downloadUrl);

    console.log(`Downloaded ${unique.length} metadata records.`);
})();
```

The CSV downloads locally. Do not commit generated CSV files to this
repository.
