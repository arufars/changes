# 🚀 Membuat Changelog dan Rilis Secara Otomatis dengan Github Release

Repositori ini memberikan konfigurasi `Membuat Changelog dan Rilis Secara Otomatis dengan Github Release` dengan alat-alat seperti `commitlint`, `commitize`n, dan `release-it` agar dapat membuat proses pengelolaan versi perangkat lunak dan pembuatan catatan perubahan menjadi lebih mudah dan otomatis.

## 📖 Instalasi
### Commitlint + Commitizen
Commitlint dan Commitizen adalah alat yang digunakan untuk memastikan bahwa commit yang dilakukan sesuai dengan aturan yang telah ditentukan. Anda dapat melihat aturan tersebut pada file `commitlint.config.js` dan `commitizen.config.js`.

### Commitlint
Anda dapat menginstal Commitlint dengan menjalankan perintah berikut:

```bash
npm install --save-dev @commitlint/config-conventional @commitlint/cli
```
### Commitizen
Anda dapat menginstal Commitizen dengan menjalankan perintah berikut:

```bash
npm install --save-dev commitizen @commitlint/cz-commitlint
```
### Release-it
Release-it adalah alat yang digunakan untuk melakukan release pada proyek Anda secara otomatis maupun manual. Kami menggunakan plugin `@release-it/conventional-changelog` untuk membuat changelog secara otomatis. Anda dapat melakukan instalasi dengan menjalankan perintah berikut:

```bash
npm install --save-dev release-it @release-it/conventional-changelog
```

## 📝 Konfigurasi

### Commitlint + Commitizen
Pertama, buat file `commitlint.config.js` pada root project dan isi dengan konfigurasi berikut:

```js
module.exports = { extends: ["@commitlint/config-conventional"] };
```
Selanjutnya, tambahkan konfigurasi pada package.json seperti di bawah ini:

```json
 "config": {
    "commitizen": {
      "path": "@commitlint/cz-commitlint"
    }
  }
```

### Release-it
Pertama, buat file `release-it.json` pada root project dan isi dengan konfigurasi berikut:

```json
 "release-it": {
    "git": {
      "changelog": "git fetch --all && git log --pretty=format:\"* %s (%h)\" `git describe --tags --abbrev=0`..HEAD",
      "commitMessage": "v${version}"
    },
    "github": {
      "release": true,
      "releaseName": "v${version}",
      "tokenRef": "GITHUB_TOKENS"
    },
    "npm": {
      "publish": false
    },
    "plugins": {
      "@release-it/conventional-changelog": {
        "infile": "CHANGELOG.md",
        "preset": {
          "name": "conventionalcommits",
          "types": [
            {
              "type": "feat",
              "section": "### ✨ Features"
            },
            {
              "type": "fix",
              "section": "### 🐛 Bug Fixes"
            },
            {
              "type": "chore",
              "hidden": true
            },
            {
              "type": "docs",
              "hidden": true
            },
            {
              "type": "style",
              "hidden": true
            },
            {
              "type": "refactor",
              "hidden": true
            },
            {
              "type": "perf",
              "hidden": true
            },
            {
              "type": "test",
              "hidden": true
            }
          ],
          "releaseCount": 1,
          "outputUnreleased": true,
          "whoCanContribute": "CONTRIBUTORS.md"
        },
        "contributors": true
      }
    }
  }
```
### Script Package.json
Tambahkan script pada `package.json` seperti dibawah ini : 

```json
{
  "scripts": {
    "commit": "git-cz",
    "release": "dotenv release-it --",
    "release:beta": "dotenv release-it -- --preRelease=beta"
  }
}
```

### Buat environment variable

```bash
GITHUB_TOKENS=github_token_punya_anda
```

### Buat .env file

```bash

touch .env

```

### 📝 Referensi
- [Commitlint]()
- [Commitizen]()
- [Release-it]()
- [Release-it Conventional Changelog]()

### 📝 Lisensi
MIT

### 📝 Author
[Arufars 👨‍💻]()