# About Me

Hey ! I'm Lilian ,an old independent blockchain security researcher currently active in audit contests on platforms such as Sherlock,Code4rena and Codehawks. Here, you can see some of my past findings and contest results.

### Contact Information
I am always looking for new opportunities be it a job opening, private audit/security review or simply for collaborations. Feel free to reach out via:

Twitter: [@lil.eth]([https://twitter.com/nasri13611])

Discord: @lil_eth

# Summary

| Overall | High risk |  Medium risk | Audited contests |
|:--:|:--:|:--:|:--:|
| 1 High/Medium | 11 High | 22 Medium | 5 Low |  

# Sherlock  <img src="https://audits.sherlock.xyz/_next/static/media/sherlock_logo.dc2b3290.svg" width=22 height=22>

This repository contains web3 vulnerabilities discovered during Sherlock or Code4rena bug bounty contests. They are organized by the protocol in which they were found.

### Audit Results

1. [Allo V2 (High: 2, Medium: 2)](#Allo_V2)
2. [Tokemak (High: 2,Medium : 0,Low : 3)](#Tokemak)
3. [Hubble Exchange (High: 2, Medium: 2)](#Hubble_Exchange)
4. [Arrakis (High: 0, Medium: 2)](#Arrakis)
5. [JOJO (High: 1, Medium: 0)](#JOJO)
6. [RealWagmi (High: 1, Medium: 2,Low : 1)](#RealWagmi)
7. [USSD (High: 2, Medium: 5)](#USSD)
8. [Y2K (High: 1, Medium: 0)](#Y2K)
9. [Derby (High: 0, Medium: 2)](#Derby)
10. [Frankencoin (High: 0, Medium: 2)](#Frankencoin)
11. [Gitcoin (High: 0, Medium: 1)](#Gitcoin)
12. [Malt (High: 0, Medium: 1)](#Malt)
13. [Telcoin (High: 0, Medium: 1)](#Telcoin)
14. [Bond (High: 0, Medium: 1)](#Bond)


# CodeHawks <img src="https://res.cloudinary.com/droqoz7lg/image/upload/v1689080263/snhkgvtsidryjdtx0pce.png" width=50 height=27>

This repository contains web3 vulnerabilities discovered during Sherlock or Code4rena bug bounty contests. They are organized by the protocol in which they were found.

### Audit Results


## Details 

## Tokemak

## Allo_V2

## Hubble_Exchange
- [Depositors might receive zero shares due to integer division](<https://github.com/nasri136/Findings_Details/blob/main/H_HubbleExchange_ZeroShareReceptionMightHappen.md>)
- [Malicious user can blocks withdraws](<https://github.com/nasri136/Findings_Details/blob/main/H_HubbleExchange_WithdrawalBreakPoint.md>)
- [Oracle.sol Assume that stablecoin Price is stable](<https://github.com/nasri136/Findings_Details/blob/main/M_HubbleExchange_StablePriceAssumption.md>)
- [latestRoundData might return stale or incorrect results](<https://github.com/nasri136/Findings_Details/blob/main/M_HubbleExchange_StalePrice.md>) 




## Arrakis
- [Initialize() allows whitelisting token0 and token1 as routers](<https://github.com/nasri136/Findings_Details/blob/main/M_Arrakis_Token0_1_WhitelistIsIneffective.md>)
- [Slot0 Usage is easily manipulable](<https://github.com/nasri136/Findings_Details/blob/main/M_Arrakis_Slot0.md>)

## JOJO

- [Loss Of Funds When Transfering](<https://github.com/nasri136/Findings_Details/blob/main/H_JOJO_LossOfFundsWhenTransfering.md>)

## RealWagmi

- [Slot0 Usage](<https://github.com/nasri136/Findings_Details/blob/main/H_RealWagmi_Slot0Usage.md>)
- [Rounding Error](<https://github.com/nasri136/Findings_Details/blob/main/M_RealWagmi_RoundingError.md>)
- [Rounding Error 2](<https://github.com/nasri136/Findings_Details/blob/main/M_RealWagmi_RoundingError2.md>)
- [Manager Exit Scam](<https://github.com/nasri136/Findings_Details/blob/main/L_RealWagmi_ManagerExit.md>)

## USSD

- [Unchecked Slippage](<https://github.com/nasri136/Findings_Details/blob/main/H_USSD_UncheckedSlippage.md>)
- [Unrestricted Minting Burning Method](<https://github.com/nasri136/Findings_Details/blob/main/H_USSD_UnrestrictedMintingBurningMethod.md>)
- [Misconfigured Oracle Address](<https://github.com/nasri136/Findings_Details/blob/main/M_USSD_Misconfigured%20Oracle%20Address.md>)
- [Misconfigured Price Feed Aggregator](<https://github.com/nasri136/Findings_Details/blob/main/M_USSD_Misconfigured%20Price%20Feed%20Aggregator.md>)
- [Outdated Price](<https://github.com/nasri136/Findings_Details/blob/main/M_USSD_OutdatedPrice.md>)
- [Rounding Error](<https://github.com/nasri136/Findings_Details/blob/main/M_USSD_RoundingError.md>)
- [Uninitialized Oracle Address](<https://github.com/nasri136/Findings_Details/blob/main/M_USSD_Uninitialized%20Oracle%20Address.md>)

## Y2K

- [Users can update and delete others' queues](<https://github.com/nasri136/Findings_Details/blob/main/H_Y2K_Users%20can%20update%20and%20delete%20others%20queues.md>)

## Derby

- [ERC20 Return Value Not Checked](<https://github.com/nasri136/Findings_Details/blob/main/M_Derby_ERC20%20Return%20Value%20Not%20Checked.md>)
- [Native Funds Can Be Lost](<https://github.com/nasri136/Findings_Details/blob/main/M_Derby_Native%20Funds%20Can%20Be%20Lost.md>)

## Frankencoin

- [Minting limit is not well affected by cl](<https://github.com/nasri136/Findings_Details/blob/main/M_Frankencoin_Minting%20limit%20is%20not%20well%20affected%20by%20cl.md>)
- [RestructureCaptable Not Well Implemented](<https://github.com/nasri136/Findings_Details/blob/main/M_Frankencoin_RestructureCaptable%20Not%20Well%20Implemented>)

## Gitcoin

- [Lack of Access Control leading to DOS](<https://github.com/nasri136/Findings_Details/blob/main/M_Gitcoin_Lack%20of%20Access%20Control%20leading%20to%20DOS.md>)

## Malt

- [PriceTargetInconsisten](<https://github.com/nasri136/Findings_Details/blob/main/M_Malt_PriceTargetInconsisten.md>)

## Telcoin

- [Front Run of addBlackList()](<https://github.com/nasri136/Findings_Details/blob/main/M_Telcoin_Front%20Run%20of%20addBlackList().md>)

## Bond

- [Bond Option Token can be created too late](<https://github.com/nasri136/Findings_Details/blob/main/M_Bond_BondsCreatedTooLate.md>)
