This page will focus on installing and configuring a privacy-focused web browsing experience.

# Which browser should I use?

Google Chromium based browsers are the most popular, and for good reason. It is a modern and fast browser project that supports a huge number of extensions and can be configured to be rather secure. Many forks are based on the project including Microsoft's own Edge browser [as of 2020](https://support.microsoft.com/en-gb/microsoft-edge/download-the-new-microsoft-edge-based-on-chromium-0f4a3dd7-55df-60f5-739f-00010dba52cf).

Unfortunately Chromium is tightly coupled with Google services and trackers. The web browsing experience itself is also not very private by default. For those reasons, I would recommend using a more privacy-oriented open-source Chromium-based browser - either [Brave](https://brave.com) or [Ungoogled Chromium](https://github.com/Eloston/ungoogled-chromium#readme).

## Brave Browser

### Pros

- **Privacy-focused by default:** *Brave Shields* are enabled by default and extends some of Chromium's privacy features (like blocking cross-site cookies) with tracker, fingerprint and ad blocking. It's configurable, you can crank it up to be aggressive, and it's built into the browser, making it fast and reducing the need for third-party extensions to do the same job.

- **Advanced features:** In addition to the standard private browsing (incognito) mode, Brave has a 'private browsing with Tor' mode and can handle `.onion` links. There's also WebTorrent, Brave Wallet, Ethereum integrations, and other features.

- **Brave Rewards:** If enabled, you can earn BAT crypto as you use your browser in exchange for receiving occasional adverts delivered via push notification. You can use it to tip creators you like, or link an Uphold or Gemini account to cash it out. You can also enable *auto-contribute* which means that any creators that have joined the *[Brave Creators](https://creators.brave.com/)* program that you regularly consume content from will automatically receive a proportional tip depending on your earnings. It's an interesting project and a nice way for creators to monetise their websites, blogs, videos, etc. For more info see [their site](https://brave.com/brave-rewards/).

- **Configurability:** The settings menu is very thorough and fleshed out. You can customise a lot within the browser, and turn off and hide any features you don't care for.

### Cons

- **Bloat:** Let's be honest, you're probably not going to be using every feature available, so you may consider Brave to be bloated compared to other choices.

- **Corporate backing:** While the project is [open-sourced](https://github.com/brave/), there is still [a company](https://brave.com/about/) behind it and there's no guarantee that they won't "sell out" or that their values will remain constant. Exhibits: [Mozilla][moz], [DuckDuckGo][ddg]. For some, the focus on crypto is already a turn-off.

[moz]: https://thepostmillennial.com/mozillas-official-blog-we-need-more-than-deplatforming "Mozilla, developers of Firefox: 'We need more than deplatforming'"

[ddg]: https://reclaimthenet.org/duckduckgo-down-ranking-russian-disinformation/ "DuckDuckGo ends neutrality, will down-rank sites “associated with Russian disinformation”"

## Ungoogled Chromium

### Pros

- **Lightweight:** The project aims to be a drop-in replacement for Chromium, but with the Google stuff stripped out. Without any major extra features, it's probably about as lightweight a version of Chromium as you can get.

- **Google-less:** All Google services are stripped out and a fail-safe system is implemented to block background requests to Google servers.

- **FOSS**: It is a fully open source and community driven project, as opposed to Brave Browser's more corporate backing. The source is available [here](https://github.com/Eloston/ungoogled-chromium).

### Cons

- **Build time:** The build script needs to download, patch and build the Chromium source. Chromium is notorious for being a big project with a lengthy compile time. For instance, it took me over 2 hours to compile Ungoogled Chromium on my laptop with an 11th-gen quad-core Intel CPU.

- **Configurability:** The settings menu is actually quite bare and some of the options you might expect from a browser are either hidden or not there. For instance, in order to clear browser data on exit, a flag must be set on the `chrome://flags` page.

- **Web Store:**  The Chrome Web Store is disabled by default. If you wish to install extensions directly from the web store, you'll need to perform a small tweak.

# Installation

TODO

# Configuration

TODO
