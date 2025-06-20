# Session 1 – Introduction to Smart Contract Auditing
## What is a Smart Contract Audit?

A **smart contract audit** is a comprehensive security review of a contract's source code and architecture to identify vulnerabilities before or after deployment. It typically includes:

* Business logic verification
* Input validation
* Access control validation
* Economic/exploit simulations
* Manual code review and test reproduction
* Use of automated tooling (static analysis, fuzzing)
* Threat modeling and adversarial thinking

Audits **reduce risk**, **build trust**, and are a **non-negotiable step** before launching production smart contracts.

## Why Audits Are Critical in Web3

Web3 is trustless, permissionless, and high-stakes. Smart contracts:

* Often holds millions to billions in value
* Cannot be patched like traditional systems
* Are exposed to a global adversarial audience 24/7

**Consequences of no/faulty audits**:

* Fund drains (flash loans, logic errors, faulty access control)
* Protocol takeovers (governance exploits)
* User trust loss and regulatory scrutiny
  ### **<ins>Disclaimer:</ins>    Doing audits isn’t just due diligence—it’s essential for every protocol.**
  

## Security Researcher vs Smart Contract Auditor

| Role        | Security Researcher                  | Smart Contract Auditor                         |
| ----------- | ------------------------------------ | ---------------------------------------------- |
| Focus       | Exploiting unknown bugs in the wild  | Preventing exploits before or after deployment |
| Process     | Open-ended investigation             | Structured, scoped, spec-aligned review        |
| Deliverable | PoCs, CVEs, writeups, bounty reports | Formal reports with severity + mitigation      |
| Environment | Deployed protocols, public targets   | Both deployed and protocols on testing phase.  |

- [Spearbit's tweet](https://x.com/spearbit/status/1681078061003952129) highlights the contrast between an auditor and a security researcher.
  ----
  ![](https://github.com/0xDelvine/Chain-check-web3clubs-sessions/blob/main/chaincheckImg/Screenshot%20From%202025-06-18%2015-13-47.png)
  ----
  ![](https://github.com/0xDelvine/Chain-check-web3clubs-sessions/blob/main/chaincheckImg/Screenshot%20From%202025-06-18%2015-14-02.png)

## Bug Bounty vs Audit

| Factor    | Bug Bounty                      | Formal Audit                             |
| --------- | ------------------------------- | ---------------------------------------- |
| Stage     | Post-deploy                     | Pre-deploy (usually)                     |
| Reviewers | Global community (crowdsourced) | Expert auditors (internal/external team) |
| Incentive | Pay-per-vuln                    | Paid engagement                          |
| Guarantee | Opportunistic coverage          | Deterministic & scoped coverage          |

### Bug Bounty/contest Platforms
1) [**Immunefi**](https://immunefi.com)
2) [**Cantina**](https://cantina.xyz)
3) [**Code4rena**](https://code4rena.com)
4) [**Codehawks**](https://codehawks.cyfrin.io/)
5) [**Sherlock**](https://audits.sherlock.xyz)
6) [**Hats Finance**](https://hats.finance)

## Vulnerability Categories

### Logic Flaws

* Misaligned reward distribution
* Input validation vulns (e.g., uncapped mint, unrestricted governance)
* Incorrect accounting/overflows/underfows with `unchecked` math

### Low-level Security Flaws

* Reentrancy (`call.value()` patterns or state-modification ordering)
* Storage collisions in proxies
* Malicious approvals (ERC20 nuances)


## Structure of a Professional Audit Report
- Audit report format vary depending on the [platform](#bug-bountycontest-platforms).

1. **Executive Summary**

   * Protocol overview, scope, methodology, findings count by severity

2. **Findings Section**

   * Each issue includes:

     * Title, severity, impacted contracts
     * Explanation of vulnerability
     * Technical PoC
     * Impact & likelihood
     * Suggested mitigation

3. **Fix Status**

   * Acknowledged / Fixed / Partially Fixed

4. **Appendices**

   *  Supplementary sections - threat models, gas optimizations, tool versions
- The above tempelate can be accessed [here.](https://github.com/0xDelvine/Chain-check-web3clubs-sessions/blob/main/chaincheckImg/BestTemplate.md)

## Roadmap to Becoming a Smart Contract Auditor

1. **Deepen Solidity + EVM Knowledge**

   * Understand storage layout, delegatecall, assembly, proxies

2. **Study Real Audits**

   * Analyze reports from various well known auditing/security researching companies
     1) Spearbit SR Company
        - Github Repo - https://github.com/spearbit/portfolio
     2) Guardian Audits
        - Github Repo - https://github.com/GuardianAudits/Audits
     3) Pashov Audit Group
        - Github Repo - https://github.com/pashov/audits
     4) Shieldfy-Security
        - Github Repo - https://github.com/shieldify-security/audits-portfolio
     5) BlockSec Team
        - Github Repo - https://github.com/blocksecteam/audit-reports
           
        
    * Analyze reports from past audit competitions and bug bounty events
      - [Solodit](https://solodit.xyz) - platform that compiles security vulnerabilities (findings) and bounties from multiple security and auditing firms in the web3 space.

3. **Reproduce Exploits**


   * Fork hacks from [rekt.news](https://rekt.news), write Foundry tests to reproduce them.
     - [DefiHackLabs](https://github.com/SunWeb3Sec/DeFiHackLabs/tree/main) has reproduced most of past DeFi hack incidents using Foundry. This helps you study scripts used then use them to try and recreate the hacks/exploits.
   * Shadow Auditing -  it's a technique where security researchers REDO a past completed contest, which already has its final report out.
      - The 2 main benefits of Shadow Auditing are:-
        * You're simulating  a real contest
        * After completing the audit, you can IMMEDIATELY read the report.
      - Perfoming a shadow audit dramatically increase your learning rate & progress and allows you to answer the following questions:
         -  what bugs you missed?
         -  why you missed them?
         -  what bugs you found and if they were described well?
         -  investigate the attack path of each finding?

4. **Tool Mastery**

   * Use Slither, surya, audit wizard etc for analysis
   * Write fuzz tests in Foundry, Hardhat etc

5. **Join Audit Contests**
  
   * Check out first flights by Cyfrin Codehawks: [codehawks first-flights](https://codehawks.cyfrin.io/first-flights)
   * Compete in audit & Bug bounty platforms Code4rena/Sherlock/Cantina
   * Learn from writing reports

  There are 2 cool sites one can use to track public web3 security audit contests:
  1.  [Daily Warden](https://www.dailywarden.com/) - shows all active and upcoming security contests.
  2.  [VigilSeek](https://www.vigilseek.com/) - like an upgrade of daily warden, it has a feature to filter contests based on language, platform e.t.c
     
### **NOTE:** 
**Check out first flights by Cyfrin Codehawks: [codehawks first-flights](https://codehawks.cyfrin.io/first-flights) - These are beginner-friendly contest that helps one sharpen their auditing skills.**
- Participate in a contest from one of the [platforms](#bug-bountycontest-platforms) mentioned above.
- Learn from writing reports.

6. **Do CTF challs**
   * [Ethernaut CTF](https://ethernaut.openzeppelin.com/) - by Openzeppelin
   * [Damn Vulnable DeFi](https://www.damnvulnerabledefi.xyz/)
   * [Capture The Ether](https://capturetheether.com/challenges/)

7. **Build a Portfolio**

   * Blog technical writeups - e.g on medium 
   * Share GitHub reviews or test repos
   * Threat posts - on X


## Case Studies
[Rekt News](https://rekt.news) is a media outlet that reports on hacks, exploits, and failures in the Web3, DeFi (Decentralized Finance), and crypto ecosystem. 
On 17th June, 2025, [Meta Pool's](https://www.metapool.app/) light was dimmed with approximately $142k worth of assets across Ethereum and bridged L2 chains (i.e., Optimism and Linea). 
### What is Meta Pool?
Meta Pool is a multi-chain liquid staking protocol that offers liquid staking tokens across networks like Ethereum, NEAR, Solana, Aurora, ICP, and more, along with features such as Vote-to-Earn through DAO governance rewards, liquidity pools for earning yield, and a restaking aggregator on Solana. 
- Checkout there [documentation](https://docs.metapool.app/) for more info.
### Exploit Summary
 Meta Pool has a staking contract that is based on OpenZeppelin’s ERC4626. It allows users to stake ETH and recieve a share token called mpETH representing their stake. This staking contract includes a  `mint()` function that the attacker exploited to mint ~9701 mpETH without providing any ETH, since it has no checks assertain that ETH was transferred before minting mpETH. This exploit led to a total loss of about 56.35 ETH across Ethereum and Layer 2s (Optimism and Linea).
 
[Here](https://docs.metapool.app/master/security/audits/ethereum) is a full post-mortem of the incident that has been completed in collaboration with Blocksec team.

