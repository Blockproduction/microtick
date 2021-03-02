# Microtick chain migration instructions

# Warning: this guide is not yet complete

These are the steps to upgrade from `microtickzone-a1` to `microtickzone-a2`. The Microtick team
will post the new genesis file, but we recommend that validator operators
use these instructions to verify genesis file. Please join us in the [validator channel of the Microtick Discord.](https://discord.gg/jz7Jt95)

If the proposal `Microtick-a2 Upgrade Proposal` passes, the target time for the upgrade procedure is
on `March 18, 2021 at or around 15:00 UTC`. Since block times vary, the precise block height will be `3,343,205`.

  - [Preliminary](#preliminary)
  - [Risks](#risks)
  - [Recovery](#recovery)
  - [Upgrade Procedure](#upgrade-procedure)
  - [Notes for Service Providers](#notes-for-service-providers)

## Preliminary

No changes have been made to the Microtick software. This upgrade is intended to reset the state of the chain,
requiring less disk space and making it easier for a node to recover. We will also add some additional markets.
This will not yet be the Stargate upgrade. Please join us in the [validator channel of the Microtick Discord.](https://discord.gg/jz7Jt95)

## Risks

As a validator performing the upgrade procedure on your consensus nodes carries a heightened risk of
equivocation (aka double-signing) and with it, the slashing penalty of 5% of your staked TICK.
The most important parts of this procedure are 1) verifying your software version and 
2) verifying the genesis file hash before your validator begins signing blocks.

The riskiest thing that an operator can do is to discover a mistake and repeat the upgrade
procedure again during the network startup. If you discover a mistake in your process, 
wait for the network to start before correcting it. If the network is halted and you have
started with a different genesis file than the expected one, get help from the Microtick team
before resetting your validator.

## Recovery

Prior to exporting the `microtickzone-a1` state, we advise operators to take a full data snapshot at the
export height before proceeding. Snapshotting depends heavily on infrastructure, but generally this
can be done by backing up the `.mtcli` and `.mtd` directories.

It is critically important to back-up the `.mtd/data/priv_validator_state.json` file after stopping your mtd process. This file is updated every block as your validator participates in a consensus rounds. This critical file is necessary to prevent double-signing, in case the upgrade fails and the previous chain needs to be restarted.

## Upgrade Procedure

__Note__: It is assumed you are currently operating a full-node running v1.0.0 of the Microtick software.

- The version/commit hash of Microtick v1.0.0: `13c5059c68a7322fa6da41d6031ebc8d3f9f575b`
- The upgrade height as agreed upon by governance: **3,343,205**

1. Verify you are currently running the correct version (v0.34.6+) of Microtick:

   ```bash
   $ mtd version --long
   name: Microtick
   server_name: mtd
   client_name: mtcli
   version: v1.0.0
   commit: 13c5059c68a7322fa6da41d6031ebc8d3f9f575b
   ```

2. Export existing state from `microtickzone-a1`:

   **NOTE**: We recommend that validator operators take a full data snapshot at the export
   height before proceeding in case the upgrade does not go as planned, or in case an insufficient
   amount of voting power comes online within an agreed upon amount of time. If we need to relaunch
   microtickzone-a1, the chain will fallback see [Recovery](#recovery) for how to proceed.

   Before exporting state via the following command, the `mtd` binary must be stopped:

   ```bash
   $ mtd export --for-zero-height --height=3343205 > mt_genesis_export.json
   ```

3. Verify the SHA256 of the (sorted) exported genesis file:

   ```bash
   $ jq -S -c -M '' mt_genesis_export.json | shasum -a 256
   [PLACEHOLDER]  mt_genesis_export.json
   ```
   
4. Verify you are still running the correct version (v1.0.0) of Microtick:

   ```bash
   $ mtd version --long
   name: Microtick
   server_name: mtd
   client_name: mtcli
   version: v1.0.0
   commit: 13c5059c68a7322fa6da41d6031ebc8d3f9f575b
   ```

5. Migrate exported state:

   ```bash
   $ mtd migrate v1.0.0 mt_genesis_export.json --chain-id=microtickzone-a2 --genesis-time=[PLACEHOLDER]> genesis.json
   ```
   
6. Verify the SHA256 hash of the final genesis JSON [(compare it with others in Discord)](https://discord.gg/jz7Jt95):

   ```bash
   $ jq -S -c -M '' genesis.json | shasum -a 256
   [PLACEHOLDER]  genesis.json
   ```

7. Reset state:

   **NOTE**: Be sure you have a complete backed up state of your node before proceeding with this step.
   See [Recovery](#recovery) for details on how to proceed.

   ```bash
   $ mtd unsafe-reset-all
   ```

8. Move the new `genesis.json` to your `.mtd/config/` directory

Please join us in the [validator channel of the Microtick Discord](https://discord.gg/jz7Jt95) to verify your genesis hash before launching.
