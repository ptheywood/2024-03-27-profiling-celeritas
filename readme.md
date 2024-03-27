# 2024-03-27 Profiling Celeritas 

Slides for talk on 2024-03-27 as part of [GridPP51 & SWIFT-HEP07](https://indico.cern.ch/event/1366954).

## Dependencies

1. [Install quarto](https://quarto.org/docs/get-started/)
2. Chromium based browser for PDF generation (if local)

## Building slides

## HTML

```bash
# Render project-configured files
quarto render
# Render by filename 
quarto render slides.qmd --to html
```

## Print to PDF

[Qaurto Print to PDF](https://quarto.org/docs/presentations/revealjs/presenting.html#print-to-pdf)

> 1. Toggle into Print View using the E key (or using the Navigation Menu)
> 2. Open the in-browser print dialog (CTRL/CMD+P).
> 3. Change the Destination setting to Save as PDF.
> 4. Change the Layout to Landscape.
> 5. Change the Margins to None.
> 6. Enable the Background graphics option.
> 7. Click Save 🎉

### Live-preview

``` bash
quarto preview slides.qmd
```

## Hosted via gh-pages

gh-action based publication via `.github/workflows/publish.yml`

See https://ptheywood.uk/2024-03-27-profiling-celeritas

Initial publication set up required:

1. Create an empty `gh-pages` branch

    ```bash
    git checkout --orphan gh-pages
    git reset --hard
    git commit --allow-empty -m "Initialising gh-pages branch"
    git push origin gh-pages
   ```

2. Ensure the github repository setting `pages > Source` has correctly set to `Branch: gh-pages`
3. Return to the `main` branch (`git checkout main`)
4. Run `quarto publish gh-pages` once to create `_publish.yml` in the gh-pages branch.
    * This is required by quarto-dev/quarto-actions/publish
5. Subsequent events which trigger `.github/workflows/publish.yml` (`workflow_dispatch`, pushes to `main`) should result in updated content.
