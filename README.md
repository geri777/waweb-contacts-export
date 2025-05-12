# Export your Whatsapp contacts in Firefox 

With pure Javascript which you enter in the Dev Console of Firefox you can export your contacts along with the last message
into a CSV File.

## Instructions

- Open Whatsapp in Firefox
- Launch Developer Console (Press F12)
- Navigate to the "Console" Tab
- Copy and paste the Javascript of script.js into the Console and press Enter

The script should no go through all your chats and export Name, Phone Number, last Message
At the end a Tab separated file is generated and downloaded, which can be used with Excel / Libre Office
The script can be stopped by entering the following command into the Console: 

```js
window.__stopWhatsappExport = true; 
```
## Note ⚠

*This script works for the GERMAN Whatsapp Version only.*
The Script uses a couple of selectors to find the contact details overlay, etc. The selectors search for titles like "Kontaktinfo", which is the German word for "Contact Details", for example
If you use any other language than German you need to modify the script, i.e. replace these words with your local translation. You are very welcome to share the modifications.

## Script

This is the script - just copy / paste it:

```js
(async () => {
  window.__stopWhatsappExport = false; // ← Cancel control

  const delay = ms => new Promise(res => setTimeout(res, ms));
  const seen = new Set();
  const output = [];

  function extractContactInfoFromDialog() {
    let name = "Unnamed";
    let number = null;

    let kontaktinfoDiv = Array.from(document.querySelectorAll("div"))
      .find(el => el.textContent?.trim() === "Kontaktinfo");

    if (!kontaktinfoDiv) return { name, number };

    const parent = kontaktinfoDiv.closest("header")?.parentElement;
    if (!parent) return { name, number };

    const siblingTexts = Array.from(parent.querySelectorAll("span.selectable-text.copyable-text"));

    for (const el of siblingTexts) {
      const text = el.textContent?.trim();
      const title = el.getAttribute("title");

      if (!number && /^\+?\d[\d\s\-().]{6,}$/.test(text)) {
        number = text.replace(/\s+/g, "").replace(/-/g, "");
      }

      if (
        name === "Unbenannt" &&
        text &&
        !text.toLowerCase().includes("kontaktinfo") &&
        !/^\+?\d[\d\s\-().]{6,}$/.test(text)
      ) {
        name = text;
      }

      if (
        name === "Unnamed" &&
        title &&
        !title.toLowerCase().includes("kontaktinfo") &&
        !/^\+?\d[\d\s\-().]{6,}$/.test(title)
      ) {
        name = title;
      }

      if (name !== "Unnamed" && number) break;
    }

    return { name, number };
  }

  console.log("📋 Starting WhatsApp-Contacts-Export (TSV)...");

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

      // Scrollen und Klick
      const outer = listitem.firstElementChild;
      const middle = outer?.firstElementChild;
      const inner = middle?.firstElementChild;
      const clickTarget = inner?.children[1];
      if (!clickTarget) continue;

      clickTarget.scrollIntoView({ behavior: "smooth", block: "center" });
      ["pointerdown", "mousedown", "mouseup", "click"].forEach(type => {
        const event = new PointerEvent(type, {
          bubbles: true,
          cancelable: true,
          composed: true,
          pointerType: "mouse",
          isPrimary: true,
        });
        clickTarget.dispatchEvent(event);
      });

      await delay(1200);

      const profileButton = document.querySelector('div[role="button"][title="Profildetails"]');
      if (!profileButton) continue;
      profileButton.click();
      await delay(1200);

      const { number } = extractContactInfoFromDialog();

      // Search field focus: as a UI focus reset.
      const searchTarget = document.querySelector('div[role="textbox"][aria-label="Sucheingabefeld"] > p.selectable-text.copyable-text');
      if (searchTarget) {
        const ev = new MouseEvent("click", { bubbles: true, cancelable: true, view: window });
        searchTarget.dispatchEvent(ev);
        await delay(500);
      }

      output.push([name, number || "", lastMessage].join("\t"));
      console.log("✔️", name, number || "keine Nummer", lastMessage || "no message");
    }

    if (!foundNew) {
      stableCycles++;
    } else {
      stableCycles = 0;
    }

    await delay(500);
  }

  if (output.length > 0) {
    const header = "Name\tNummer\tLast Message";
    const blob = new Blob([header + "\n" + output.join("\n")], { type: "text/tab-separated-values" });
    const a = document.createElement("a");
    a.href = URL.createObjectURL(blob);
    a.download = "whatsapp_kontakte_export.tsv";
    a.click();
    console.log(`✅ TSV-Export finished: ${output.length} contacts saved.`);
  } else {
    console.error("❌ Sorry, not a single contact could be extracted.`);
  }
})();
```

# Disclaimer

This script comes without any warranty - if you like it, give it a star ★

This work is not affiliated with, endorsed by, or in any way officially connected with WhatsApp Inc., Facebook Inc., or any of their subsidiaries or affiliates. The name "WhatsApp" as well as related names, marks, emblems, and images are registered trademarks of their respective owners.
