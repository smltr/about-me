# FindServers.net Deep Dive

## Project Overview

  **Context:**
    - Personal passion project, built because I wanted it to exist for my own use
    - Live at findservers.net
    - Counter-Strike 2 server browser that's faster and more feature-rich than the built-in one or other third party options

  **Stack:**
    - Backend: Go web server, Redis for caching
    - Frontend: React (single HTML file, no build process)
    - Automation: GitHub Actions for scheduled updates
    - Deployment: fly.io

  **Why?:**
    - CS2's built-in server browser is slow and limited, a lot of spam servers
    - Not many ways to filter server list
    - Innacurate info, duplicate entries
    - Third party options bloated with ads, show only a few servers at a time, and are missing the "feel" of scrolling through thousands of active servers

## Technical Implementation

  Data Flow

  GitHub Actions (Cron) → API Refresh → Steam Web API → Data Processing → Redis Cache

  Client API → Serves Frontend → Frontend requests server list → Server pulls from redis and responds

  **1. Caching & Performance**
    - **Challenge:** Steam API is slow, can't query it for every user request
    - **Solution:**
      - Redis caching layer stores processed server data
      - Automated refresh every 5 minutes via GitHub Actions
      - Fallback refresh on user request if data is stale
    - **Benefits:** Sub-second response times, reduced API calls
    - May eventually transition to relational db to track server data over time, but for now storing it all as a blob is very quick and easy
      - (or may use both)

  **2. Frontend Performance**
    - **Challenge:** Rendering thousands of servers causes browser slowdown
      - long time to load initially, slow when scrolling & clicking
    - **Solution:**
      - Custom virtual scrolling implementation intercepting mousewheel events
      - Only render visible servers + buffer
    - **Trade-off:** Lost smooth scrolling animation, but gained performance, simple to implement, and entire server list remains on front end for quick filtering

  ```javascript
    // virtual scrolling

    // state for virtual scrolling
    const [servers, setServers] = useState([]);
    const [scrollPosition, setScrollPosition] = useState(0);
    const [viewportHeight, setViewportHeight] = useState(0);

    const ITEM_HEIGHT = 27; // server rows are static sizes

    // how many items do we need
    const viewportItemCount = Math.max(20, Math.floor(viewportHeight / ITEM_HEIGHT));


    const maxScrollPosition = Math.max(0, filteredServers.length - viewportItemCount);

    // key concept, instead of scrolling pixels, we scroll array indices
    function renderVisibleItems() {
      const startIndex = scrollPosition;
      const endIndex = scrollPosition + viewportItemCount;

      return servers
        .slice(startIndex, endIndex)  // only render visible slice
        .map((server, index) => <ServerRow key={server.address} server={server} />)
    }

    const SCROLL_DISTANCE = 3;
    // intercept wheel events and convert to index changes
    function onScroll(event) {
      if (event.deltaY > 0) {
        // scroll down: increase index
        setScrollPosition(prev => Math.min(prev + SCROLL_DISTANCE, maxScrollPosition));
      } else {
        // scroll up: decrease index
        setScrollPosition(prev => Math.max(prev - SCROLL_DISTANCE, 0));
      }
    }

    // Scrollbar interaction
    function onScrollbarClick(event) {
      const scrollbar = event.currentTarget;
      const clickY = event.clientY - scrollbar.getBoundingClientRect().top;
      const scrollbarHeight = scrollbar.getBoundingClientRect().height;

      // Convert click position to scroll index
      const clickPercent = clickY / scrollbarHeight;
      const newScrollPosition = Math.round(clickPercent * maxScrollPosition);

      setScrollPosition(Math.max(0, Math.min(newScrollPosition, maxScrollPosition)));
    }
  ```

  **3. Filters**
    - **Problem:** Built-in browser can't do negative filters or complex queries
    - **Solution:**
      - Both positive and negative filter syntax: `map: de_, -dust2`
      - Real-time filtering on frontend
    - **User Experience:** Instant results, powerful search capabilities

## Impact

  - Solved my own problem - I actually use this regularly
  - Better UX than existing alternatives
  - I plan to spread the word and hopefully get more people playing on community servers, can also use feedback

## What This Project Demonstrates

  **Entrepreneurial Thinking:**
    - Identified and solved a real problem I experienced
    - End-to-end product ownership
    - Ability to follow through from concept to a working app

  **Pragmatic Engineering:**
    - Chose simple solutions over complex ones
    - Focused on core functionality over perfect architecture
    - Made smart trade-offs between features and maintainability
      - makeshift scrolling solution: allowed me to use and test app now
      - Up and running with no front end build steps

Next, deep dive into LanguageLabs -> [languagelabs.md](languagelabs.md)
