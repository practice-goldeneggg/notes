# Terraform
[Terraform](http://www.terraform.io/)

===

## インストール
* 公式からバイナリ落とす
    * __Golang製で、様々なプラットフォーム向けにクロスコンパイルしたバイナリが提供されている__


### インストール for Developer using Golang
[hashicorp/terraform](https://github.com/hashicorp/terraform)

#### 前提条件
* Go入れとく

#### ビルド

```
% go get github.com/mitchellh/gox  # クロスコンパイルツール
% cd $GOPATH/src/github.com
% mkdir hashicorp
% cd hashicorp
% git clone https://github.com/hashicorp/terraform.git
% cd terraform
% make updatedeps  # ココでコケる！

# github.com/mitchellh/go-libucl
In file included from ../../mitchellh/go-libucl/object.go:5:
./go-libucl.h:4:10: fatal error: 'ucl.h' file not found
#include <ucl.h>
         ^
1 error generated.
make: *** [updatedeps] Error 2


% make
```
