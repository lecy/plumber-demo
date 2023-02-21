demo data file created by: 

```r
bmf <- read.csv( "https://nccs-data.urban.org/dl.php?f=bmf/2022/bmf.bm2208.csv" )
tinybmf <- dplyr::sample_n( bmf, 10000 )
write.csv( tinybmf, "tinybmf.csv", row.names=F )
```
