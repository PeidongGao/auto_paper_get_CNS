# Science Metadata Export SOP

Target page: <https://www.science.org/journal/science/research>

1. Review [the repository disclaimer](../DISCLAIMER.md) and the publisher's
   current terms before use.
2. Open the target page.
3. Open the browser Web Inspector and select the Console.
4. Set `maxPages` to a small number, then paste and run the script once.

```javascript
(async () => {
    const maxPages = 5;
    const rows = [];

    for (let page = 1; page <= maxPages; page++) {
        const startPage = (page - 1) * 20;
        const pageUrl =
            `https://www.science.org/journal/science/research?startPage=${startPage}&ConceptID=505154`;
        const html = await fetch(pageUrl).then(response => response.text());
        const doc = new DOMParser().parseFromString(html, "text/html");

        Array.from(doc.querySelectorAll('a[href*="/doi/10.1126/science."]'))
            .forEach(link => {
                const text = link.parentElement?.parentElement
                    ?.innerText.trim().replace(/\s+/g, " ") ||
                    link.innerText.trim();
                rows.push({
                    page,
                    startPage,
                    type: text.toLowerCase().includes("review")
                        ? "Review"
                        : "Research Article",
                    title: link.innerText.trim(),
                    url: link.href,
                    doi: link.href.replace("https://www.science.org/doi/", ""),
                    text
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
        "page,startPage,type,title,url,doi,text",
        ...unique.map(row =>
            `"${escapeCSV(row.page)}","${escapeCSV(row.startPage)}","${escapeCSV(row.type)}","${escapeCSV(row.title)}","${escapeCSV(row.url)}","${escapeCSV(row.doi)}","${escapeCSV(row.text)}"`
        )
    ].join("\n");

    const timestamp = new Date().toISOString().slice(0, 16)
        .replace("T", "_")
        .replace(":", "-");
    const blob = new Blob([csv], { type: "text/csv;charset=utf-8" });
    const downloadUrl = URL.createObjectURL(blob);
    const anchor = document.createElement("a");
    anchor.href = downloadUrl;
    anchor.download = `science_${timestamp}_pages_1-${maxPages}.csv`;
    document.body.appendChild(anchor);
    anchor.click();
    anchor.remove();
    URL.revokeObjectURL(downloadUrl);

    console.log(`Downloaded ${unique.length} metadata records.`);
})();
```

The CSV downloads locally. Do not commit generated CSV files to this
repository.
