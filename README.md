# Description

This is v0.1 spec for a lightweight protocol which allows embedded slave/peripheral bus devices to describe their configuration to the drivers. Mainly focused for [AMBA](https://en.wikipedia.org/wiki/Advanced_Microcontroller_Bus_Architecture) open standard and its AHB/APB/AIX buses.

| Register       | Width (bits) |Access   | Required | Alignment |
|----------------|--------------|---------|----------|-----------|
|ID_SIGNATURE    |16            |Read-only|Yes       |Hard coded by the protocol |
|MODE            |8             |Read-only|Yes       |Hard coded by the protocol |
|VENDOR_ID       |16            |Read-only|Optional  |Given by the MODE register |
|DEVICE_ID       |8             |Read-only|Optional  |Given by the MODE register |
|CONFIG_REVISION |8             |Read-only|Optional  |Given by the MODE register |
|<CUSTOM_BLOCK>    |Up to the user|Read-only|Optional, but recommended|Given by the MODE register |


The first two registers are aligned to a 64-bit address bus with a 8-bit data bus, which allows it to be compatible with any embedded target bus. The rest of the registers will be aligned depending on the value of the MODE register. 

After implementing these 2-5 registers the layout, the <CUSTOM_BLOCK> configuration registers are up-to the vendor/user and their schema/layout can be whatever is suiting. Even when the <CUSTOM_BLOCK> configuration registers are not enforced by the protocol, there would be little added value to implement this protocol without taking advantage of the <CUSTOM_BLOCK>.

## ID_SIGNATURE
By default it's 0x1DB8, a 16-bit value is considered as sufficient as this protocol is not meant to crawl the address space, as that could take too long on a small embedded device, reach unimplemented regions, cause exceptions, or have undesired effects on other peripherals which have side-effects on the read-only accesses. The driver is expected to be provided with the base address of the peripheral and this signature is used to do sanity check if this protocol can be used.

## MODE

| Field | Offset (bits) | Width (bits) | Description |
|-------|--------|-------|-------------|
|MODE_DATA_WIDTH |0 |2 | 0=8-bit, 1=16-bit, 2=32-bit, 3=64-bit
|MODE_ALIGNMENT |2 |2 | 0=8-bit, 1=16-bit, 2=32-bit, 3=64-bit
|MODE_SIMPLIFIED |4 |1 | 0=The optional registers have to be implemented, 1=Omit optional registers |
|RESERVED | 5 | 3 | Currently not used

The MODE_DATA_WIDTH >= MODE_ALIGNMENT condition needs to be meet, it's invalid to have a 64-bit data width peripheral with 8-bit alignment. 

When MODE_SIMPLIFIED is set to 1, then VENDOR_ID, DEVICE_ID and CONFIG_REVISIONs have to be omitted.

## VENDOR_ID

Make pull-request, to reserve own vendor/user IDs (one vendor/user can allocate multiple IDs). However because I do not expect this spec to be widely used there is a allocated region of valid reserved VENDOR_IDs which are grand for experiments, these IDs can be freely used, just there is no guarantee that they will not be used by somebody else.

| ID | Vendor/User |
|----|--------|
| 0x0- 0xffef | free to allocate|
| 0xfff0 - 0xffff | Allocated for experimental use |

## DEVICE_ID

It's up to the vendor/user how his devices will be recognized, hash or incremental ID can be used.
If the vendor has more than 255 of devices using this protocol, then multiple Vendor IDs can be used.

## CONFIG_REVISION

In case the same device has multiple different configurations then CONFIG_REVISION can be used. Similar to [semantic versioning](https://semver.org/), only updating major version when API changes, CONFIG_REVISION should be incremented when different layout/schema of the config is used. So driver can detect these changes and probe the config with the correct schema. If for some reason the config did not change, but the hardware/driver behaviour did change, then user is free to add their own additional REVISION register into the <CUSTOM_BLOCK>. This should allow one single firmware/driver to dynamically on runtime change implementations depending what revisions are detected.