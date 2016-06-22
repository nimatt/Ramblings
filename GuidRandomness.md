# .NET Guids, how random are they?

I keep seeing _Sytem.Guid_ beeing used as a random token for accessing everything from
files to password reset pages. They consist of 16 bytes which corresponds to roughly
3.4e+38 values, which should be more than enough. If they are random, but exactly
how random are they. They are not intended to be used as random tokens but I
haven't managed to find any information on just how random the are. So I figured that
I can at least do a quick check on some of its properties when I was stuck on a train
for two hours anyway.

*DISCLAMER*: When reading this, you should take into account that I am only scratching
the surface and that my knowledge in statistics leave a lot to wish for.

## Get Data

First of all, we need to get some data to run our tests on. For this I decided to
use PowerShell
```PowerShell
$lines = (1..1000000) | % {[string]::Join(',', [Guid]::NewGuid().ToByteArray())}
$lines | Out-File -Encoding utf8 -FilePath 'guidData.csv'
```
This will give us a csv file where each column is one byte of the Guid. Unfortunately
PowerShell save the file in UTF8 with BOM which R doesn't expect but that was
easily fixed just opening and resaving the file using Notepad++.

When we've gotten that far, we can load the data into R using
```R
data <- read.csv(file='F:/git/Lab/GuidTest/guidData.csv', header = FALSE)
```

## Frequency plot
So on with the interesting stuff. The first thing I thought I would check is how
the distributions looked using a simple histogram plot.

```R
par(mfrow=c(4,4))
for (i in 1:16) {
  hist(data[[i]], breaks=c(-0.5:255.5), freq = FALSE, main=paste("Byte ", i-1, sep=''), xlab='Value')
}
```

![histogram plot](https://raw.githubusercontent.com/nimatt/Ramblings/master/images/guidRandomness/hist.png)

Most of it looks just the way we want it to, nice and flat. But what is up with
byte 7 and 8? This can also be seen when plotting all bytes added together.

![histogram plo all](https://raw.githubusercontent.com/nimatt/Ramblings/master/images/guidRandomness/histAll.png)

## Pearson's Chi-squared
Next I wanted to know if any of the bytes had dependencies between them. One way
to test independence is to do a so called Peason's Chi-squared test. Which is to
quote Wikipedia

>  a statistical test applied to sets of categorical data to evaluate how likely it is that any observed difference between the sets arose by chance.

The following snippet runs the test to check the independence between the bytes.
```R
for (i in 1:16) {
  for (j in i:16) {
    if (i != j) {
      res <- chisq.test(table(data[[i]],data[[j]]))
      if (res$p.value < 0.1) {
        print(c(i, j, res$p.value))
      }
    }
  }
}
```
This prints all the cases where the probability goes below 10%.

| Byte | Byte | p-value |
|--------|--------|---------|
| 0| 4| 0.0109|
| 0| 11| 0.0270|
| 1| 9| 0.0266|
| 1| 10| 0.0402|
| 1| 11| 0.0050|
| 1| 15| 0.0916|
| 2| 7| 0.0197|
| 4| 11| 0.0758|
| 5| 14| 0.0848|
| 7| 20| 0.0829|

They do not seem very independant...

Then what about sequences? Are a Guid dependent on the previous ones? Lets do the
same test but with shifted data instead.

```R
for (i in 1:10) {
  for (j in i:16) {
    shifted_data <- data[0:-i,j]
    res <- chisq.test(table(data[1:length(shifted_data),j], shifted_data))
    if (res$p.value < 0.1) {
      print(c(i, j, res$p.value))
    }
  }
}
```
Here we try to shift the data from 1 to 10 steps in each of the bytes and once
more print cases below 10%.

|Shift|Byte|p-value|
|---|---|---|
| 1| 14| 0.0447|
| 2| 2| 0.0210|
| 3| 7| 0.0429|
| 4| 9| 0.0213|
| 4| 10| 0.0250|
| 4| 11| 0.0445|
| 6| 8| 0.0003|
| 6| 15| 0.0739|
| 9| 8| 0.0991|
| 9| 9| 0.0978|
| 10| 10| 0.0691|

## Fast Fourier Transform (FFT)
The result from the shifting made me wonder if it is possible to find some cyclic
properties in any of the bytes (even if they where not created at fixed intervals).
I tried using the *stats* package in R to plot the result from a FFT. This turned
out to give me a few completely useless plots before my train arrived and I shut
down the computer. So I therefore leave this to the interested reader (please let
  me know how it goes).

## Conclusion
This doesn't give any solid proof that a Guid is possible to predict or force break.
But it does give strong indications that they are not as random as they look. But hey,
why use the when you have the *System.Security.Cryptography.RNGCryptoServiceProvider*?
