```shell
find . -name '*.ipynb' -print -exec grep 'DataReader' {} \;   
```
```shell
find . -name '*.ipynb' -exec grep -l "mapper" {} \;
```
