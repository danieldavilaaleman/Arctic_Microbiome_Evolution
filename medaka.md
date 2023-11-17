# Installation of medaka
```
module load biobuilds/2017.11
module load python/3.10.4
virtualenv medaka --python=python3 --prompt "(medaka) "
. medaka/bin/activate
pip install --upgrade pip
pip install medaka
```
To install samtools whitout root access, I need to set a prefix for installation on ~/usr directory
```
wget https://github.com/samtools/samtools/releases/download/1.18/samtools-1.18.tar.bz2
tar -xvjf samtools-1.18.tar.bz2
cd samtools-1.18
./configure --prefix=$HOME/usr
make
make install
```
Same thing with bcftool and htslib
