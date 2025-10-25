## Node.js Dependencies
- Install latest `nvm`

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/master/install.sh | bash
```

- Use `nvm` to install latest version of `node.js`

```bash
nvm install node
```

- With the latest version of `node` installed, use `apt` to install `npm`

```bash
sudo apt install -y npm
```

## Gemini
- Get Google API key from vault

```bash
npm install -g @google/gemini-cli@latest
```

- Run

```bash
gemini
```

## Codex
- Get OpenAI API key from vault

```bash
npm install -g @openai/codex
```

- Run

```bash
codex
```


