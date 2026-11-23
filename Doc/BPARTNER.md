# Table BPARTNER (Tiers / Business Partner)

* **Table:** `BPARTNER`
* **Abbreviation:** `BPR`
* **Module:** Données de base (Common Data)
* **Description:** Central master table storing shared general data for all business partners (customers, suppliers, carriers, prospects, sales reps, and miscellaneous partners) before role-specific extensions.

---

## Physical Data Model (ER Diagram)

```mermaid
erDiagram
    %% Core Business Partner Table
    BPARTNER {
        string BPRNUM PK "BPR0: Code tiers"
        string BPRNAM "Raison sociale"
        string BPRSHO "Intitulé court (BPR1)"
        string CRY FK "Pays -> TABCOUNTRY"
        string LAN FK "Langue -> TABLAN"
        string CUR FK "Devise -> TABCUR"
        string BPAADD FK "Adresse par défaut -> BPADDRESS"
        string CNTNAM FK "Contact par défaut -> CONTACT"
        string BIDNUM FK "RIB par défaut -> BID"
        string BPRFCT "Fonction / Rôle"
        int BPCFLG "Flag Client"
        int BPSFLG "Flag Fournisseur"
        int BPTFLG "Flag Transporteur"
        int BPRFLG "Flag Prospect"
        int REPFLG "Flag Représentant"
        string CRN "Numéro SIREN / ID"
        string NAF "Code NAF"
        string EECNUM "Numéro de TVA intracom."
        int BETFCY "Inter-site (BPR2)"
        string FCY FK "Site associé -> FACILITY (BPR2)"
        int ENAFLG "Actif (Oui/Non)"
    }

    %% Role Extension Tables (1-to-0..1)
    BPCUSTOMER {
        string BPCNUM PK,FK "BPC0: Clients"
        string BPCNAM "Nom client"
        string BCGCOD FK "Catégorie client"
        string BPCINV FK "Client facture"
        string BPCPYR FK "Client payeur"
    }
    BPSUPPLIER {
        string BPSNUM PK,FK "BPS0: Fournisseurs"
        string BPSNAM "Nom fournisseur"
        string BSGCOD FK "Catégorie fournisseur"
        string BPSINV FK "Fournisseur facturant"
        string BPSPAY FK "Fournisseur payé"
    }
    BPCARRIER {
        string BPTNUM PK,FK "BPT0: Transporteurs"
    }
    SALESREP {
        string REPNUM PK,FK "REP0: Représentants"
    }

    %% Shared Sub-Entities (Addresses, Contacts, Bank Details)
    BPADDRESS {
        int BPATYP PK "BPA0: Type entité (1=Tiers)"
        string BPANUM PK,FK "Code tiers"
        string BPAADD PK "Code adresse"
        string CRY FK "Pays"
    }
    CONTACT {
        int BPATYP PK "CNT0: Type entité (1=Tiers)"
        string BPANUM PK,FK "Code tiers"
        string CNTNAM PK "Code contact"
    }
    BID {
        int BPATYP PK "BID0: Type entité (1=Tiers)"
        string BPANUM PK,FK "Code tiers"
        string BIDNUM PK "N° compte bancaire / IBAN"
    }

    %% Reference & Setup Tables
    TABCOUNTRY {
        string CRY PK "TCY0: Pays"
    }
    TABCUR {
        string CUR PK "TCU0: Devises"
    }
    TABLAN {
        string LAN PK "TLA0: Langues"
    }
    FACILITY {
        string FCY PK "FCY0: Sites (Inter-site)"
    }

    %% Role Relationships (Shared Primary Key BPRNUM)
    BPARTNER ||--o| BPCUSTOMER : "BPRNUM = BPCNUM"
    BPARTNER ||--o| BPSUPPLIER : "BPRNUM = BPSNUM"
    BPARTNER ||--o| BPCARRIER : "BPRNUM = BPTNUM"
    BPARTNER ||--o| SALESREP : "BPRNUM = REPNUM"

    %% Sub-Entity Relationships (BPATYP = 1)
    BPARTNER ||--o{ BPADDRESS : "BPANUM (BPATYP=1)"
    BPARTNER ||--o{ CONTACT : "BPANUM (BPATYP=1)"
    BPARTNER ||--o{ BID : "BPANUM (BPATYP=1)"
    BPADDRESS ||--o{ CONTACT : "BPATYP+BPANUM"
    BPADDRESS ||--o{ BID : "BPATYP+BPANUM"

    %% Lookup Relationships
    TABCOUNTRY ||..o{ BPARTNER : "CRY"
    TABCUR ||..o{ BPARTNER : "CUR"
    TABLAN ||..o{ BPARTNER : "LAN"
    FACILITY ||..o{ BPARTNER : "FCY (BETFCY)"
```

---

## Indexes & Keys

| Key | Columns | Duplicates | Description |
| :--- | :--- | :--- | :--- |
| **BPR0** | `BPRNUM` | No | Primary key (Business Partner code) |
| **BPR1** | `BPRSHO` | Yes | Short description search key |
| **BPR2** | `BETFCY + FCY + BPRNUM` | No | Inter-site / Facility partner lookup |

---

## Core Fields Summary

| Column | Description | Type | Linked Table / Expression |
| :--- | :--- | :--- | :--- |
| `BPRNUM` | Business partner code | `BPR` | `[BPR]BPR0=[BPR]BPRNUM` |
| `BPRNAM` | Company / Partner name | `NAM` | — |
| `BPRSHO` | Short name | `SHO` | — |
| `CRY` | Country | `CRY` | `[TCY]TCY0=[BPR]CRY` |
| `LAN` | Language | `LAN` | `[TLA]TLA0=[BPR]LAN` |
| `CUR` | Currency | `CUR` | `[TCU]TCU0=[BPR]CUR` |
| `BPAADD` | Default address | `ADR` | `[BPA]BPA0=1;[BPR]BPRNUM;[BPR]BPAADD` |
| `CNTNAM` | Default contact | `AIN` | `[CNT]CNT0=1;[BPR]BPRNUM;[BPR]CNTNAM` |
| `BIDNUM` | Default bank ID | `BID` | `[BID]BID0=1;[BPR]BPRNUM;[BPR]BIDNUM` |
| `BPCFLG` | Customer flag | `M (1)` | `1 = No, 2 = Yes` (`BPCUSTOMER`) |
| `BPSFLG` | Supplier flag | `M (1)` | `1 = No, 2 = Yes` (`BPSUPPLIER`) |
| `BPTFLG` | Carrier flag | `M (1)` | `1 = No, 2 = Yes` (`BPCARRIER`) |
| `BETFCY` | Inter-site flag | `M (1)` | `1 = No, 2 = Yes` |
| `FCY` | Associated site | `FCY` | `[FCY]FCY0=[BPR]FCY` |
| `CRN` | Company registration no. (SIREN) | `CRN` | — |
| `EECNUM` | EU VAT number | `EEC` | — |
