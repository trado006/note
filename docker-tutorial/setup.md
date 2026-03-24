### Re-install dependencies if change

Nội dung file bash script kiểm tra có sự thay đổi từ file quản lý dependencies

**Cách 1**: Kiểm tra bằng timestampt

```sh
#!/bin/sh
set -e

if [ ! -d "node_modules" ]; then
  echo "📦 node_modules not found → installing dependencies..."
  npm ci
elseif if [ package.json -nt node_modules ]; then
  echo "📦 package.json is newer than node_modules → reinstalling dependencies..."
  npm ci
else
  echo "✅ node_modules exists → skipping install"
fi

exec "$@"
```

**Cách 2**: Kiểm tra bằng hash content bị thay đổi

```sh
#!/bin/sh
set -e

HASH_FILE=".deps_hash"
CURRENT_HASH=$(cat package.json package-lock.json 2>/dev/null | md5sum | awk '{print $1}')

if [ ! -d "node_modules" ] || [ ! -f "$HASH_FILE" ] || [ "$CURRENT_HASH" != "$(cat $HASH_FILE)" ]; then
  echo "📦 Installing dependencies..."
  rm -rf node_modules
  npm ci
  echo "$CURRENT_HASH" > $HASH_FILE
fi

exec "$@"
```

