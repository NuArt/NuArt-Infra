# Incident: GitHub HTTPS auth selhal při clone

## Co se stalo
Clone přes HTTPS s heslem selhal — GitHub od 2021 nepřijímá heslo pro git operace.

## Fix
SSH klíč: `~/.ssh/id_ed25519` na Docker VM, registrovaný na GitHubu jako „NuArt Docker Server".
Clone přes `git@github.com:...`.

## Poučení
Na server vždy klonovat přes SSH (`git@github.com:`), ne HTTPS.
Tohle repo `nuart-infra` proto pushovat přes SSH remote.
