# Cell Metadata Export SOP

Target page: <https://www.cell.com/cell/newarticles>

1. Review [the repository disclaimer](../DISCLAIMER.md) and the publisher's
   current terms before use.
2. Open the target page.
3. Open the browser Web Inspector and select the Console.
4. Paste and run the script below once.

```javascript
(async () => {
    const rows = [];
    const html = await fetch("https://www.cell.com/cell/newarticles")
        .then(response => response.text());
    const doc = new DOMParser().parseFromString(html, "text/html");

    const links = Array.from(doc.querySelectorAll('a[href*="/fulltext/"]'))
        .filter(link => link.innerText.trim() !== "Full-Text HTML");

    links.forEach(link => {
        const title = link.innerText.trim();
        const url = link.href;
        const articleNode =
            link.closest("li") ||
            link.parentElement?.parentElement ||
            link.parentElement;
        const text = articleNode
            ? articleNode.innerText.trim().replace(/\s+/g, " ")
            : title;
        rows.push({ title, url, text });
    });

    const seen = new Set();
    const unique = rows.filter(row => {
        if (seen.has(row.url)) return false;
        seen.add(row.url);
        return true;
    });

    const escapeCSV = value =>
        String(value ?? "").replaceAll('"', '""');
    const csv = [
        "title,url,text",
        ...unique.map(row =>
            `"${escapeCSV(row.title)}","${escapeCSV(row.url)}","${escapeCSV(row.text)}"`
        )
    ].join("\n");

    const timestamp = new Date().toISOString().slice(0, 16)
        .replace("T", "_")
        .replace(":", "-");
    const blob = new Blob([csv], { type: "text/csv;charset=utf-8" });
    const downloadUrl = URL.createObjectURL(blob);
    const anchor = document.createElement("a");
    anchor.href = downloadUrl;
    anchor.download = `cell_${timestamp}.csv`;
    document.body.appendChild(anchor);
    anchor.click();
    anchor.remove();
    URL.revokeObjectURL(downloadUrl);

    console.log(`Downloaded ${unique.length} metadata records.`);
})();
```

The CSV downloads locally. Do not commit generated CSV files to this
repository.
