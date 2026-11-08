# TODO – needforheat-server-configuration

Open work for this repository, distilled from the old NeedForHeat Trello board, including the discussions on the cards (paraphrased and regrouped) and from the 2026 maintenance review.

Items that touch several repositories are written out in full in one *lead* repository; the others list them with a link (→). Keep the details up to date in the lead repository only.

## Now: needed to run at home this winter

- [ ] <a id="backup"></a>**Back up the NeedForHeat database.** Include the MariaDB database of API v3 in the backup and test a restore. Also affects: needforheat-server-api. *(Trello #654, #1159)*
- [ ] → Verify the server stays up under real load — see [needforheat-server-api](https://github.com/energiegedrag-nl/needforheat-server-api/blob/main/TODO.md#down) *(Trello #1155)*

## Soon: robustness and correctness

- [ ] <a id="deploy"></a>**Make deployment reproducible.** Document and automate the deployment (compose files, Traefik, secrets) so a new server can be set up from this repository; let Portainer reuse its existing volume. *(Trello #1029, #751, #1133)*
- [ ] → Configurable API path prefix; move new firmware to /v3 — see [needforheat-generic-firmware](https://github.com/energiegedrag-nl/needforheat-generic-firmware/blob/main/TODO.md#prefix)

## Later: features and ideas

- [ ] <a id="traefik"></a>**Certificate handling in Traefik.** Set the ACME contact address and check renewal after a Let's Encrypt revocation. *(Trello #1132, #306)*
- [ ] → Finish the rename from Twomes to NeedForHeat — see [needforheat-generic-firmware](https://github.com/energiegedrag-nl/needforheat-generic-firmware/blob/main/TODO.md#rename) *(Trello #1030)*
- [ ] → CONTRIBUTING.md with the branching workflow — see [needforheat-generic-firmware](https://github.com/energiegedrag-nl/needforheat-generic-firmware/blob/main/TODO.md#contrib) *(Trello #394, #1032)*
