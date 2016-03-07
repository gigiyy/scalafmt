# scalafmt
> Any style guide written in English is either so brief that it’s ambiguous, or
> so long that no one reads it.
>
> -- Bob Nystrom, ["Hardest Program I've Ever Written"][journal],
>    Dart Team, Google.

[![Maven Central](https://maven-badges.herokuapp.com/maven-central/com.geirsson/scalafmt-core_2.11/badge.svg)](https://maven-badges.herokuapp.com/maven-central/com.geirsson/scalafmt-core_2.11)
[![codecov.io](https://codecov.io/github/olafurpg/scalafmt/coverage.svg?branch=master)](https://codecov.io/github/olafurpg/scalafmt?branch=master)
[![Build Status](https://travis-ci.org/olafurpg/scalafmt.svg?branch=master)](https://travis-ci.org/olafurpg/scalafmt)
[![Join the chat at https://gitter.im/olafurpg/scalafmt](https://badges.gitter.im/olafurpg/scalafmt.svg)](https://gitter.im/olafurpg/scalafmt?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)


## [Documentation](docs)
Head to https://olafurpg.github.io/scalafmt for installation instructions.

[docs]: https://olafurpg.github.io/scalafmt

### Contributing

* `core/testOnly org.scalafmt.FormatTests`: runs the fast, most relevant, unit tests.
    * After each run, a performance report is generated in `target/index.html`. 
    I usually keep a browser tab open at `localhost:3000/target/index.html`
    along with this background process:
    `browser-sync start --server --files "target/*.html"`.
    See [Browsersync](https://www.browsersync.io/).
* `core/testOnly org.scalafmt.FidelityTest`: runs the formatter on all files in
  this project.
* `run-benchmarks.sh` script to run jmh benchmarks.
* `core/test:runMain  org.scalafmt.FormatExperimentApp`: clones Scala.js, runs
  formatter on all cloned files and prints summary.
* instructions for the tests [are here](core/src/test/resources).

### Updates (mostly for myself)

* March 3st
  * Created [Intellij plugin][intellij].
  * Published release candidate to Maven Central.
* March 1st
  * See [scalafmt formatted with scalafmt][format-with-scalafmt]. Still have a few
    things to fix before merging that branch.
  * Can format all files in Scala.js!
```
Scala.js
========
Success: 920
Total: 920
+------------+--------+--------------+----------+---------+---------+----------+
|         Max|     Min|           Sum|      Mean|       Q1|       Q2|        Q3|
+------------+--------+--------------+----------+---------+---------+----------+
|8.406,451 ms|2,713 ms|172.574,826 ms|187,581 ms|32,167 ms|79,585 ms|173,553 ms|
+------------+--------+--------------+----------+---------+---------+----------+

JMH
===
# Run complete. Total time: 00:03:24

Benchmark                        Mode  Cnt     Score     Error  Units
GenJsCode.scalafmt               avgt   10  1375.404 ± 203.097  ms/op
GenJsCode.scalariform            avgt   10   238.604 ±  10.338  ms/op
JavaLangString.scalafmt          avgt   10   181.791 ±   3.683  ms/op
JavaLangString.scalariform       avgt   10     6.878 ±   0.769  ms/op
OptimizerCore.scalafmt           avgt   10  1330.836 ±  74.754  ms/op
OptimizerCore.scalariform        avgt   10   235.918 ±  10.049  ms/op
ScalaJSClassEmitter.scalafmt     avgt   10   305.206 ±  44.357  ms/op
ScalaJSClassEmitter.scalariform  avgt   10    37.935 ±   4.010  ms/op

Note. JavaLangString didn't terminate before.
```

* Feb 26th
  * Able to format 916 of 920 files in Scala.js! And 75 percentile format in
  under 200ms.
```
Benchmark                        Mode  Cnt     Score    Error  Units
GenJsCode.scalafmt               avgt   10  1283.002 ± 77.631  ms/op
GenJsCode.scalariform            avgt   10   228.591 ± 10.815  ms/op
OptimizerCore.scalafmt           avgt   10  1242.236 ± 46.238  ms/op
OptimizerCore.scalariform        avgt   10   230.396 ±  8.253  ms/op
ScalaJSClassEmitter.scalafmt     avgt   10   308.831 ± 15.678  ms/op
ScalaJSClassEmitter.scalariform  avgt   10    35.309 ±  0.602  ms/op

+-------------+--------+--------------+----------+--------+---------+----------+
|          Max|     Min|           Sum|      Mean|      Q1|       Q2|        Q3|
+-------------+--------+--------------+----------+--------+---------+----------+
|15.362,715 ms|2,165 ms|309.145,298 ms|340,094 ms|29,87 ms|74,412 ms|197,866 ms|
+-------------+--------+--------------+----------+--------+---------+----------+
```
* Feb 25th
  * Can format [GenJsCode](https://github.com/scala-js/scala-js/blob/master/compiler/src/main/scala/org/scalajs/core/compiler/GenJSCode.scala)
    in ~1.5s! Previously didn't terminate.
  * Still struggling with other pathological cases [like here][emitter].
* Feb 23th
  * replaced memoization with a simple heuristic to prune out dead states and
  got up to ~100x speed improvements.
  * fixed remainig output bugs, no more AST changes or other formatting errors.
[emitter]: https://github.com/scala-js/scala-js/blob/29e46c2284afd48dcfe2dcacbcfe9f9fdbadf617/tools/shared/src/main/scala/org/scalajs/core/tools/linker/backend/emitter/ScalaJSClassEmitter.scala#L584
```
JMH
===
[info] Benchmark                    Mode  Cnt     Score    Error  Units
[info] BaseLinker.scalafmt          avgt   10   134.951 ± 65.239  ms/op
[info] BaseLinker.scalariform       avgt   10    26.815 ±  6.342  ms/op
[info] Division.scalafmt            avgt   10   169.684 ± 15.607  ms/op
[info] Division.scalariform         avgt   10    35.659 ±  6.397  ms/op
[info] OptimizerCore.scalafmt       avgt   10  1389.404 ± 88.609  ms/op
[info] OptimizerCore.scalariform    avgt   10   256.030 ± 33.611  ms/op
[info] SourceMapWriter.scalafmt     avgt   10    34.418 ±  0.940  ms/op
[info] SourceMapWriter.scalariform  avgt   10     7.728 ±  0.679  ms/op

NOTE. OptimizerCore is 4k LOC and did not format under 1min before today.

Scala.js
========
Formatter timed out: 29
Success: 891

Total: 920
+-------------+--------+--------------+----------+---------+---------+----------+
|          Max|     Min|           Sum|      Mean|       Q1|       Q2|        Q3|
+-------------+--------+--------------+----------+---------+---------+----------+
|10.008,617 ms|3,029 ms|425.785,847 ms|477,874 ms|35,239 ms|95,027 ms|342,221 ms|
+-------------+--------+--------------+----------+---------+---------+----------+

Note. These timing results are ad-hoc benchmarks, expect up to a ±100ms margin
error. Still, observe that 75% of files in the Scala.js repo format in under
350ms.
```

* Feb 21th
  * Scala.js experiment with 10s timeout. Now, formats 848 files out of 920! 
  
```
org.scalafmt.Error$FormatterOutputDoesNotParse: 9
scala.meta.ParseException: 19
org.scalafmt.Error$FormatterChangedAST$: 4
java.lang.ClassCastException: 11
org.scalafmt.Error$CantFormatFile$: 9
Success: 848
Timed out: 20

+------------+--------+--------------+----------+---------+----------+----------+
|         Max|     Min|           Sum|      Mean|       Q1|        Q2|        Q3|
+------------+--------+--------------+----------+---------+----------+----------+
|9.982,152 ms|1,343 ms|488.630,586 ms|576,215 ms|34,954 ms|114,927 ms|468,503 ms|
+------------+--------+--------------+----------+---------+----------+----------+
```

* Feb 19th
  * Ran `scalafmt` on  Scala.js repo with 10s timeout. Performance is slower
  than yesterday since 
  I fixed some formatting bugs which made the performance slightly worse.
```
Unknown Failures: 60
Timeout Failures: 21
Parse exceptions: 19
Format successes: 821
Summary of running times (in ms, 1E8 is 100ms):
Q1=25th percentile, Q2=median, Q3=75th percentile
+------+------+-------+------+------+------+------+
|   Max|   Min|    Sum|  Mean|    Q1|    Q2|    Q3|
+------+------+-------+------+------+------+------+
|9.79E9|1.65E6|5.41E11|6.58E8|3.45E7|1.19E8|5.18E8|
+------+------+-------+------+------+------+------+

Benchmark                        Mode  Cnt     Score     Error  Units
BaseLinker.scalafmt              avgt    3   749.977 ± 289.092  ms/op
BaseLinker.scalametaParser       avgt    3     8.171 ±   8.260  ms/op
BaseLinker.scalariform           avgt    3    15.634 ±  22.799  ms/op
Basic.scalafmt                   avgt    3    25.089 ±  29.384  ms/op
Basic.scalametaParser            avgt    3    18.783 ±  23.687  ms/op
Basic.scalariform                avgt    3     0.666 ±   1.207  ms/op
Division.scalafmt                avgt    3  2765.606 ± 521.940  ms/op
Division.scalametaParser         avgt    3    16.796 ±   9.724  ms/op
Division.scalariform             avgt    3    35.425 ±   9.046  ms/op
JsDependency.scalafmt            avgt    3    57.497 ±  41.198  ms/op
JsDependency.scalametaParser     avgt    3     1.792 ±   5.853  ms/op
JsDependency.scalariform         avgt    3     3.828 ±  10.027  ms/op
Semantics.scalafmt               avgt    3    60.998 ±  72.010  ms/op
Semantics.scalametaParser        avgt    3     2.053 ±   4.469  ms/op
Semantics.scalariform            avgt    3     3.954 ±   2.780  ms/op
SourceMapWriter.scalafmt         avgt    3   218.982 ±  82.800  ms/op
SourceMapWriter.scalametaParser  avgt    3     3.997 ±  11.506  ms/op
SourceMapWriter.scalariform      avgt    3     7.576 ±   7.467  ms/op
Utils.scalafmt                   avgt    3    74.701 ±  76.110  ms/op
Utils.scalametaParser            avgt    3     2.364 ±   4.006  ms/op
Utils.scalariform                avgt    3     4.812 ±   6.279  ms/op
```
* Feb 18th
  * Ran `scalafmt` on  Scala.js repo with 200ms timeout.
```
Unknown Failures: 48
Timeout Failures: 268
Parse exceptions: 13
Format successes: 592
```
  * Updated benchmark:

```
Benchmark                        Mode  Cnt     Score     Error  Units
BaseLinker.scalafmt              avgt    3   707.864 ±  87.023  ms/op
BaseLinker.scalametaParser       avgt    3    10.876 ±  49.694  ms/op
BaseLinker.scalariform           avgt    3    15.764 ±   3.908  ms/op
Basic.scalafmt                   avgt    3    25.363 ±  46.914  ms/op
Basic.scalametaParser            avgt    3    19.608 ±  18.905  ms/op
Basic.scalariform                avgt    3     0.659 ±   1.250  ms/op
Division.scalafmt                avgt    3  2253.869 ± 556.383  ms/op
Division.scalametaParser         avgt    3    16.818 ±  11.662  ms/op
Division.scalariform             avgt    3    35.331 ±  42.856  ms/op
JsDependency.scalafmt            avgt    3    51.908 ±  21.290  ms/op
JsDependency.scalametaParser     avgt    3     1.810 ±   5.086  ms/op
JsDependency.scalariform         avgt    3     3.741 ±   8.991  ms/op
Semantics.scalafmt               avgt    3    64.453 ± 107.310  ms/op
Semantics.scalametaParser        avgt    3     2.105 ±   4.223  ms/op
Semantics.scalariform            avgt    3     4.192 ±   4.279  ms/op
SourceMapWriter.scalafmt         avgt    3   203.994 ± 146.662  ms/op
SourceMapWriter.scalametaParser  avgt    3     4.099 ±  11.523  ms/op
SourceMapWriter.scalariform      avgt    3     9.539 ±  14.685  ms/op
Utils.scalafmt                   avgt    3    71.421 ± 123.586  ms/op
Utils.scalametaParser            avgt    3     2.414 ±   4.647  ms/op
Utils.scalariform                avgt    3     4.779 ±   2.180  ms/op
```
  
* Feb 16th
  * `scalafmt` can format 235 out of 917 files from Scala.js in under 200ms.
  * Ran benchmarks against Scalariform. I'm not surprised `scalafmt` is
    **significantly** slower since 1. I've done little optimizations and 2.
    `scalafmt` supports column-width limit which introduces a huge overhead.
    
```
# Run complete. Total time: 00:10:29

Benchmark                    Mode  Cnt     Score     Error  Units
BaseLinker.scalafmt          avgt   10  6226.143 ±  97.964  ms/op
BaseLinker.scalariform       avgt   10    13.604 ±   0.273  ms/op
Basic.scalafmt               avgt   10    20.116 ±   0.723  ms/op
Basic.scalariform            avgt   10     0.481 ±   0.087  ms/op
Division.scalafmt            avgt   10  6792.488 ± 519.506  ms/op
Division.scalariform         avgt   10    34.708 ±   8.885  ms/op
JsDependency.scalafmt        avgt   10   127.133 ±   6.151  ms/op
JsDependency.scalariform     avgt   10     3.395 ±   0.171  ms/op
SourceMapWriter.scalafmt     avgt   10   494.667 ±  45.247  ms/op
SourceMapWriter.scalariform  avgt   10     6.887 ±   0.078  ms/op
Utils.scalafmt               avgt   10   156.653 ±   2.729  ms/op
Utils.scalariform            avgt   10     4.214 ±   0.096  ms/op
```
* Feb 12th
  * Ran scalafmt on all [scala-js][scalajs] files with a 200ms timeout.
  The formatter completed before timeout on 68 files and timed out on 848 files.
* Feb 9rd
  * [pre-release][releases]
* Feb 3rd
  * [Sneak peek output from the
  formatter](https://htmlpreview.github.io/?https://github.com/olafurpg/scalafmt/blob/htmlpreview/reports/scalafmt-feb3.html).
  The red regions are spots where the formatter's performance still suffers.
  * The actual implementation, under `src/main/scala/`, is 684 LOC, excluding blank lines and comments and including some debugging stuff.
  * Running time for a ~500 LOC file is around 5 seconds. Will hopefully
  reach <100ms soon. Last week the formatter choked on 10 LOC (exponential
  growth, yay).
  * There is one style to begin with, adapted from the [Scala.js coding
  style](https://github.com/scala-js/scala-js/blob/master/CODINGSTYLE.md).
  Support for
  configuration flags comes later.
  * The formatter uses [scala.meta's](https://github.com/scalameta/scalameta)
  tokens and AST. The implementation works mostly on the token level, but
  relies heavily on the AST for "smarter" features.
  * Still haven't covered some common cases (like breaking before `.` in
  `a.b(c)`), so the output looks weird in some cases.

[format-with-scalafmt]: https://github.com/olafurpg/scalafmt/compare/format-with-scalafmt?expand=1
[scalajs]: https://github.com/scala-js/scala-js
[journal]: http://journal.stuffwithstuff.com/2015/09/08/the-hardest-program-ive-ever-written/
[intellij]: https://plugins.jetbrains.com/plugin/8236?pr=
[releases]: https://github.com/olafurpg/scalafmt/releases
