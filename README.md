# Export Claude for Chrome Sidebar as a Complete PDF

[中文](https://github.com/lingwsh/Export-Claude-for-Chrome-Sidebar-as-a-Complete-PDF/blob/main/README_CN.md)

**A robust, semi-automated workaround to perfectly export your Claude for Chrome side panel conversations to PDF without losing CSS styling or suffering from print pagination truncation.**

## 🛑 The Problem
The official Claude for Chrome extension lacks a native export feature. Attempting to force a print or save the HTML fails due to three main roadblocks:
1. **Side Panel Restrictions:** Chrome's strict security sandbox prevents calling `window.print()` directly inside an extension's side panel context.
2. **Execution Context Isolation:** Standard bookmarklets or injected scripts cannot penetrate the `chrome-extension://` environment.
3. **Print Truncation:** The extension uses `height: 100vh` and internal scrolling. Printers only read physical height, causing multi-page conversations to be abruptly cut off after the first page.

## 💡 The Solution
This script clones the fully rendered DOM and CSS into an unrestricted `about:blank` tab. Crucially, it injects an aggressive `@media print` stylesheet that destroys all height restrictions and scroll locks, forcing the browser to render every single page perfectly. 

To avoid copying and pasting this code every time, we utilize **Chrome DevTools Snippets** for a quick, two-click workflow.

---

## 🛠️ One-Time Setup (Automation via Chrome Snippets)

Instead of manually pasting the code every time, save it inside Chrome's built-in Snippets manager.

1. Open the Claude sidebar and right-click on a blank area (not the input box), then select **Inspect**.
2. In the DevTools window, navigate to the **Sources** tab at the top.
3. On the left pane, click the **Snippets** tab (if you don't see it, click the `>>` icon to reveal hidden tabs).
4. Click **+ New snippet** and name it something like `Claude-to-PDF`.
5. Copy and paste the following JavaScript code into the main editor window:

```javascript
// 1. Open a completely new, unrestricted blank tab
let printWindow = window.open('', '_blank');

// 2. Clone the sidebar's DOM and CSS, injecting print-reset styles
printWindow.document.write(`
  <html>
    <head>
      <base href="${location.origin}${location.pathname}">
      ${document.head.innerHTML}
      <style>
        /* Ultimate fix for print truncation: force all elements to release height and allow overflow */
        @media print {
            * {
                overflow: visible !important;
                max-height: none !important;
                box-shadow: none !important;
            }
            html, body, #root, main, div {
                height: auto !important;
                position: static !important;
            }
            /* Hide unnecessary UI elements like the bottom input box */
            fieldset, form, textarea {
                display: none !important;
            }
        }
      </style>
    </head>
    <body>
      ${document.body.innerHTML}
    </body>
  </html>
`);

// 3. Close the document write stream to ensure the DOM is fully parsed
printWindow.document.close();

// 4. Wait 2 seconds for external CSS and fonts to load, then trigger the native print dialog
setTimeout(() => {
    printWindow.print();
}, 2000);
```
6. Press `Ctrl + S` (Windows/Linux) or `Cmd + S` (Mac) to save the snippet.

---

## 🚀 How to Use It Daily

Now that the script is permanently saved in your browser, exporting takes just seconds:

1. Right-click the Claude sidebar and click **Inspect**.
2. Press `Ctrl + P` (or `Cmd + P` on Mac), type `!` to open the Snippets menu, and select `Claude-to-PDF`. 
   *(Alternatively, if you are already on the Sources tab, just right-click your snippet and select **Run**, or press `Ctrl + Enter`)*.
3. A new tab will automatically open, clone your conversation, and pop up the native Print dialog after 2 seconds.

### ⚠️ Crucial Print Settings
When the Print window appears:
* Set the Destination to **Save as PDF**.
* Expand **More settings**.
* **You MUST check "Background graphics"**. If you skip this, the dark/light mode background colors, code blocks, and chat bubbles will disappear, ruining the formatting.
