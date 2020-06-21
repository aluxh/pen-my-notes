# R notes

- [Knit RMD files via commandline](#Knit-RMD-files-via-commandline)

## Knit RMD files via commandline

The command to knit via Rstudio is

```r
rmarkdown::render('test.Rmd', 'html_document')
```

If you have added a specific format in the YAML, that you wish to knit into, you can just omit the `html_document`.

```r
rmarkdown::render("test.Rmd")
```

Then, for command line command:

```bash
Rscript -e 'library(rmarkdown);rmarkdown::render("/path/to/test.Rmd")'
```

**Note:** If you face issues *with pandoc version 1.12.3 or higher is required and was not found*, you can try the solution from [here](https://stackoverflow.com/questions/28432607/pandoc-version-1-12-3-or-higher-is-required-and-was-not-found-r-shiny). I basically use 

```bash
export RSTUDIO_PANDOC="/c/Program Files/RStudio/bin/pandoc/"
```
