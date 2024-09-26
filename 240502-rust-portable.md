---
title: Setup a portable rust on Windows
description: A solution of installing a portable rust on Windows
last_modified_at: 2024-09-26T21:30:40+09:00
---
# Setup a portable rust on Windows
✨ Created on 2024/5/2 &nbsp;&nbsp;&nbsp;&nbsp; ✨ Updated on 2024/9/26

## Main Procedure
### 1. Download rust from the [rust page](https://forge.rust-lang.org/infra/other-installation-methods.html#standalone-installers).  
For example:  
https://static.rust-lang.org/dist/rust-1.77.2-x86_64-pc-windows-gnu.tar.gz

### 2. Extract the .tar.gz file.  
For example:
```batch
set from=C:\rust-1.77.2-x86_64-pc-windows-gnu
```

### 3. Create a new folder for your portable rust.  
For example:
```batch
mkdir C:\rust
set to=C:\rust
```

### 4. Copy the folders from extracted folder to target rust folder.
```batch
xcopy /e /y %from%\cargo\* %to%\
xcopy /e /y %from%\rustc\* %to%\
xcopy /e /y %from%\rust-mingw\* %to%\
xcopy /e /y %from%\rust-std-x86_64-pc-windows-gnu\* %to%\
rem (optional) rustfmt
xcopy /e /y %from%\rustfmt-preview\* %to%\
```

### 5. Add rust to path.
```batch
set path=%to%\bin;%path%
```

## Additional Information
To solve the `can't load standard library from sysroot` error of [rust-analyzer](https://marketplace.visualstudio.com/items?itemName=rust-lang.rust-analyzer).
### 1. Download source code of rust from the [rust page](https://forge.rust-lang.org/infra/other-installation-methods.html#source-code).  
For example:  
https://static.rust-lang.org/dist/rustc-1.77.2-src.tar.xz

### 2. Create the folder
```batch
mkdir %to%\lib\rustlib\src
```

### 3. Extract downloaded file to `%to%\lib\rustlib\src`

### 4. Change extracted folder name from `rustc-1.77.2-src` to `rust`

[🚗BACK](/README.html)
