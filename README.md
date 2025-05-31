# Export your Whatsapp contacts in Chrome 

With pure Javascript which you enter in the Dev Console of Chrome you can export your contacts along with the last message
into a CSV File.

## Instructions

- Open Whatsapp in Chrome
- Launch Developer Console (Press F12)
- Navigate to the "Console" Tab
- Copy and paste the Javascript of script.js into the Console and press Enter

The script should no go through all your chats and export Name (Phone Number), last Message
At the end a Tab separated file is generated and downloaded, which can be used with Excel / Libre Office
The script can be stopped by entering the following command into the Console: 

```js
window.__stopWhatsappExport = true; 
```

## Script

This is the script - just copy / paste it:

```js
(async () => {
  window.__stopWhatsappExport = false;

  const delay = ms => new Promise(res => setTimeout(res, ms));
  const seen = new Set();
  const output = [];

  function extractContactInfoFromDialog() {
    let name = "Unnamed";
    let number = null;

    let contactInfoDiv = Array.from(document.querySelectorAll("div"))
      .find(el => el.textContent?.trim() === "Contact info" || el.textContent?.trim() === "Info kontak");

    if (!contactInfoDiv) return { name, number };

    const parent = contactInfoDiv.closest("header")?.parentElement;
    if (!parent) return { name, number };

    const siblingTexts = Array.from(parent.querySelectorAll("span.selectable-text.copyable-text"));

    for (const el of siblingTexts) {
      const text = el.textContent?.trim();
      const title = el.getAttribute("title");

      if (!number && /^\+?\d[\d\s\-()]{6,}$/.test(text)) {
        number = text.replace(/\s+/g, "").replace(/-/g, "").replace(/[()]/g, "");
      }

      if (
        name === "Unnamed" &&
        text &&
        !text.toLowerCase().includes("contact info") &&
        !text.toLowerCase().includes("info kontak") &&
        !/^\+?\d[\d\s\-()]{6,}$/.test(text)
      ) {
        name = text;
      }

      if (
        name === "Unnamed" &&
        title &&
        !title.toLowerCase().includes("contact info") &&
        !title.toLowerCase().includes("info kontak") &&
        !/^\+?\d[\d\s\-()]{6,}$/.test(title)
      ) {
        name = title;
      }

      if (name !== "Unnamed" && number) break;
    }

    return { name, number };
  }

  console.log("📋 Starting WhatsApp Contacts Export (TSV)...");

  // Reset UI state by focusing on search bar
  const searchBar = document.querySelector('div[role="textbox"][aria-label="Search input textbox"]');
  if (searchBar) {
    searchBar.click();
    await delay(500);
  }

  let stableCycles = 0;

  while (stableCycles < 5) {
    const chatList = Array.from(document.querySelectorAll("div[role='grid'] > div[role='listitem']"));
    let foundNew = false;

    for (const listitem of chatList) {
      if (window.__stopWhatsappExport) {
        console.warn("⛔️ Export canceled by user");
        break;
      }

      const name = listitem.querySelector('span[dir="auto"][title]')?.getAttribute("title")?.trim() || "Unnamed";
      const lastMessage = listitem.querySelector('span[dir="ltr"]')?.textContent?.trim().replace(/\s+/g, " ") || "";

      const key = `${name}|${lastMessage}`;
      if (seen.has(key)) continue;

      seen.add(key);
      foundNew = true;

      const clickTarget = listitem.querySelector('div[role="button"]') || listitem;
      if (!clickTarget) continue;

      clickTarget.scrollIntoView({ behavior: "smooth", block: "center" });
      await delay(500);

      clickTarget.click();
      await delay(2000);

      // Updated profile button selector
      const profileButton = [...document.querySelectorAll('div[role="button"]')]
        .find(btn => {
          const title = btn.title?.trim().toLowerCase();
          const ariaLabel = btn.getAttribute("aria-label")?.toLowerCase();
          return (
            title?.includes("profile") ||
            title?.includes("info") ||
            ariaLabel?.includes("profile") ||
            ariaLabel?.includes("info") ||
            btn.getAttribute("data-icon")?.includes("info")
          );
        });

      let contactData = { name, number: "" };
      if (profileButton) {
        profileButton.click();
        await delay(1500);
        contactData = extractContactInfoFromDialog();
      }

      // Use number if available, otherwise fallback to name (from chat list)
      const displayName = contactData.number || name;
      output.push([displayName, lastMessage].join("\t"));
      console.log("✔️", displayName, lastMessage || "no message");

      // Close contact info panel by clicking search bar
      if (searchBar) {
        searchBar.click();
        await delay(500);
      }
    }

    stableCycles = foundNew ? 0 : stableCycles + 1;
    await delay(500);
  }

  if (output.length > 0) {
    const header = "Name\tLast Message";
    const blob = new Blob([header + "\n" + output.join("\n")], { type: "text/tab-separated-values" });
    const a = document.createElement("a");
    a.href = URL.createObjectURL(blob);
    a.download = "whatsapp_contacts_export.tsv";
    document.body.appendChild(a);
    a.click();
    a.remove();
    console.log(`✅ Export complete: ${output.length} contacts saved.`);
  } else {
    console.error("❌ No contacts were extracted.");
  }
})();
```

# Disclaimer

This script comes without any warranty - if you like it, give it a star ★

This work is not affiliated with, endorsed by, or in any way officially connected with WhatsApp Inc., Facebook Inc., or any of their subsidiaries or affiliates. The name "WhatsApp" as well as related names, marks, emblems, and images are registered trademarks of their respective owners.
