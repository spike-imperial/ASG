The Ubuntu version of the latest ASG release can be downloaded from
[https://github.com/spike-imperial/ASG/releases](https://github.com/spike-imperial/ASG/releases).
We recommend that you copy it to somewhere in your PATH (for example,
/usr/local/bin/).

The ASG solver has the following dependencies:

1. [Clingo 5.3.0](https://github.com/potassco/clingo/releases). ILASP
   users should note that unlike ILASP, which only requires the Clingo
   executable, the ASG solver requires the Clingo library `libclingo`.
   We therefore recommend that you follow the installation instructions
   available at [https://potassco.org/](https://potassco.org/).
2. [ILASP 3.4.0 or later](http://www.ilasp.com). Both ILASP and Clingo 5
   must be somewhere in your PATH (for example, /usr/local/bin/).

After copying the ASG solver to your PATH, you will be able to call it
by running `ASG --mode=run --depth=[int] file.asg` for any ASG file and
integer depth `[int]`. There are several examples of valid ASG files and
LASG learning tasks
[here](https://github.com/spike-imperial/ASG/tree/master/data/).
