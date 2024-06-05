---
title: Setup a portable rust on Windows
description: A solution of installing a portable rust on Windows
last_modified_at: 2024-05-02
---
# Setup a portable rust on Windows
✨ Created on 2024/5/2

### 1. Download rust from URL:  
https://static.rust-lang.org/dist/rust-1.77.2-x86_64-pc-windows-gnu.tar.gz

### 2. Extract the .tar.gz file. For example:
```batch
set from=C:\rust-1.77.2-x86_64-pc-windows-gnu
```

### 3. Create a new folder for your portable rust. For example:
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
```

### 5. Add rust to path.
```batch
set path=%to%\bin;%path%
```

[🚗BACK](/)
