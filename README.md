# danielle
patty's attempt at a TAWS website (no link yet)

## Developing

Once you've created a project and installed dependencies with `bun install`, start a development server:

```sh
bun run dev
```

## Cloudflare Pages

Set the `SKIP_DEPENDENCY_INSTALL` variable to `1`, and your build command should be `bun install && bun run build`