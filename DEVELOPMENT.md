# DEVELOPMENT.md

## Creating a new countdown folder (checklist)

1. **Create a new subdirectory**
   ```
   countdowns/<new-countdown-slug>/
   ```

2. **Copy the baseline dashboard**
   - Copy `multi-countdown_fixed3.html`
   - Rename it to:
     ```
     <new-countdown-slug>/index.html
     ```

3. **Add data**
   ```
   <new-countdown-slug>/data/
     your_data_file.json
   ```

4. **Edit `APP_CONFIG` at the top of index.html**
   - Set:
     - `countdownName`
     - `dataFile` (relative path)
     - `mode` (`cover` or `standard`)
     - `metrics` (ordered list)

5. **Add link to root landing page**
   - Edit `/index.html`
   - Add the new countdown to the list

6. **Test locally**
   ```
   python3 -m http.server
   ```
   Visit:
   ```
   http://localhost:8000/<new-countdown-slug>/
   ```

That’s it. No build step, no parameters, no shared state.
