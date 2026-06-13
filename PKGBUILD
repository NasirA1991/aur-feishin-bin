name: Deploy to AUR

on:
  schedule:
    - cron: '0 */6 * * *'
  workflow_dispatch:

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Install dependencies
        run: sudo apt-get update && sudo apt-get install -y jq curl coreutils

      - name: Get latest release
        id: release
        run: |
          set -e

          JSON=$(curl -s https://api.github.com/repos/jeffvli/feishin/releases/latest)

          VERSION=$(echo "$JSON" | jq -r '.tag_name')
          CLEAN_VERSION="${VERSION#v}"

          echo "version=$CLEAN_VERSION" >> $GITHUB_OUTPUT

      - name: Check update
        id: check
        run: |
          set -e

          CURRENT=$(grep -Po '^pkgver=\K.*' PKGBUILD)
          LATEST="${{ steps.release.outputs.version }}"

          echo "current=$CURRENT"
          echo "latest=$LATEST"

          if [ "$CURRENT" = "$LATEST" ]; then
            echo "update=false" >> $GITHUB_OUTPUT
            exit 0
          fi

          echo "update=true" >> $GITHUB_OUTPUT

      - name: Update PKGBUILD + SHA
        if: steps.check.outputs.update == 'true'
        run: |
          set -e

          VERSION="${{ steps.release.outputs.version }}"

          # reset pkgver and pkgrel
          sed -i "s/^pkgver=.*/pkgver=$VERSION/" PKGBUILD
          sed -i "s/^pkgrel=.*/pkgrel=1/" PKGBUILD

          URL="https://github.com/jeffvli/feishin/releases/download/v${VERSION}/Feishin-linux-x64.tar.xz"

          curl -L -o feishin.tar.xz "$URL"

          SHA256=$(sha256sum feishin.tar.xz | awk '{print $1}')

          # overwrite properly formatted array
          sed -i "s|^sha256sums=.*|sha256sums=('$SHA256')|" PKGBUILD

      - name: Clean workspace
        run: |
          rm -f feishin.tar.xz || true
          git clean -fdx

      - name: Deploy to AUR
        if: steps.check.outputs.update == 'true'
        uses: KSXGitHub/github-actions-deploy-aur@v3
        with:
          pkgname: feishin-bin
          pkgbuild: ./PKGBUILD
          assets: PKGBUILD
          SSH_PRIVATE_KEY: ${{ secrets.AUR_SSH_PRIVATE_KEY }}
          commit_username: ${{ secrets.AUR_COMMIT_USERNAME }}
          commit_email: ${{ secrets.AUR_COMMIT_EMAIL }}
          updpkgsums: false
          allow_empty_commits: false
