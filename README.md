# backstage-workshop

On Ubuntu 22.04:
```bash
./scripts/setup.sh
cd ~
npx @backstage/create-app@latest --skip-install
# accept all defaults
cd backstage
yarn install
# edit app-config.yaml
yarn dev
```
