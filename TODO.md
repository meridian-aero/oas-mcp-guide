# TODO: Marketplace Migration

When ready to distribute via a Lakeside AI marketplace:

1. Create repo `meridian-aero/claude-plugins` with `.claude-plugin/marketplace.json`
2. Add `oas-mcp-guide` as a relative-source plugin entry in the manifest
3. Tag releases with semver (`v1.0.0`, `v1.1.0`, etc.)
4. For team auto-discovery, add `extraKnownMarketplaces` to project `.claude/settings.json`
5. For staged rollout, point `stable` and `latest` branches to different versions
6. Submit to official Anthropic marketplace at `claude.ai/settings/plugins/submit`
