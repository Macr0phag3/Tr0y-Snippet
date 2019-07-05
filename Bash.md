# function
## `rm`? just wait...
```
function rm(){
    array=("${(@s/ /)@}")
    for i in ${array[@]}; do

        mv $i ~/.Trash/${i##*/}"-"`date +"%Y_%m_%d-%H_%M_%S"`

    done
}
```

# alias