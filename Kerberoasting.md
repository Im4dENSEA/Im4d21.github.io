<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Kerberoasting - Schémas Détaillés</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }
        
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            padding: 20px;
            min-height: 100vh;
        }
        
        .container {
            max-width: 1400px;
            margin: 0 auto;
        }
        
        h1 {
            text-align: center;
            color: white;
            margin-bottom: 30px;
            font-size: 2.5em;
            text-shadow: 2px 2px 4px rgba(0,0,0,0.3);
        }
        
        .tabs {
            display: flex;
            gap: 10px;
            margin-bottom: 20px;
            justify-content: center;
        }
        
        .tab-button {
            padding: 15px 30px;
            background: rgba(255,255,255,0.2);
            border: 2px solid white;
            color: white;
            font-size: 1.1em;
            font-weight: bold;
            cursor: pointer;
            border-radius: 10px;
            transition: all 0.3s;
            backdrop-filter: blur(10px);
        }
        
        .tab-button:hover {
            background: rgba(255,255,255,0.3);
            transform: translateY(-2px);
        }
        
        .tab-button.active {
            background: white;
            color: #667eea;
        }
        
        .schema-container {
            display: none;
            background: white;
            border-radius: 20px;
            padding: 40px;
            box-shadow: 0 20px 60px rgba(0,0,0,0.3);
        }
        
        .schema-container.active {
            display: block;
            animation: fadeIn 0.5s;
        }
        
        @keyframes fadeIn {
            from { opacity: 0; transform: translateY(20px); }
            to { opacity: 1; transform: translateY(0); }
        }
        
        .schema-title {
            text-align: center;
            color: #667eea;
            font-size: 2em;
            margin-bottom: 40px;
            border-bottom: 3px solid #667eea;
            padding-bottom: 15px;
        }
        
        .step {
            margin-bottom: 40px;
            position: relative;
        }
        
        .step-header {
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            color: white;
            padding: 15px 20px;
            border-radius: 10px;
            font-size: 1.3em;
            font-weight: bold;
            margin-bottom: 20px;
            display: flex;
            align-items: center;
            gap: 15px;
        }
        
        .step-number {
            background: white;
            color: #667eea;
            width: 40px;
            height: 40px;
            border-radius: 50%;
            display: flex;
            align-items: center;
            justify-content: center;
            font-size: 1.2em;
            font-weight: bold;
        }
        
        .flow-diagram {
            display: flex;
            align-items: center;
            justify-content: space-between;
            margin: 20px 0;
            flex-wrap: wrap;
            gap: 20px;
        }
        
        .actor {
            background: linear-gradient(135deg, #f093fb 0%, #f5576c 100%);
            color: white;
            padding: 20px;
            border-radius: 15px;
            min-width: 180px;
            text-align: center;
            box-shadow: 0 5px 15px rgba(0,0,0,0.2);
            flex: 1;
        }
        
        .actor.attacker {
            background: linear-gradient(135deg, #fa709a 0%, #fee140 100%);
        }
        
        .actor.kdc {
            background: linear-gradient(135deg, #30cfd0 0%, #330867 100%);
        }
        
        .actor.ad {
            background: linear-gradient(135deg, #a8edea 0%, #fed6e3 100%);
            color: #333;
        }
        
        .actor.service {
            background: linear-gradient(135deg, #ff9a9e 0%, #fecfef 100%);
            color: #333;
        }
        
        .actor-title {
            font-weight: bold;
            font-size: 1.2em;
            margin-bottom: 10px;
        }
        
        .actor-icon {
            font-size: 2.5em;
            margin-bottom: 10px;
        }
        
        .arrow {
            font-size: 2em;
            color: #667eea;
            margin: 0 10px;
        }
        
        .arrow-down {
            text-align: center;
            font-size: 2em;
            color: #667eea;
            margin: 10px 0;
        }
        
        .message-box {
            background: #f0f4ff;
            border-left: 5px solid #667eea;
            padding: 15px;
            margin: 15px 0;
            border-radius: 5px;
            font-family: 'Courier New', monospace;
        }
        
        .info-box {
            background: #fff3cd;
            border-left: 5px solid #ffc107;
            padding: 15px;
            margin: 15px 0;
            border-radius: 5px;
        }
        
        .danger-box {
            background: #f8d7da;
            border-left: 5px solid #dc3545;
            padding: 15px;
            margin: 15px 0;
            border-radius: 5px;
        }
        
        .success-box {
            background: #d4edda;
            border-left: 5px solid #28a745;
            padding: 15px;
            margin: 15px 0;
            border-radius: 5px;
        }
        
        .code-box {
            background: #1e1e1e;
            color: #d4d4d4;
            padding: 20px;
            border-radius: 10px;
            font-family: 'Courier New', monospace;
            overflow-x: auto;
            margin: 15px 0;
        }
        
        .highlight {
            background: #fffacd;
            padding: 2px 5px;
            border-radius: 3px;
            font-weight: bold;
        }
        
        .defense-grid {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
            gap: 20px;
            margin: 20px 0;
        }
        
        .defense-card {
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            color: white;
            padding: 25px;
            border-radius: 15px;
            box-shadow: 0 5px 15px rgba(0,0,0,0.2);
        }
        
        .defense-card h3 {
            font-size: 1.4em;
            margin-bottom: 15px;
            border-bottom: 2px solid rgba(255,255,255,0.3);
            padding-bottom: 10px;
        }
        
        .defense-card ul {
            list-style: none;
            padding-left: 0;
        }
        
        .defense-card li {
            padding: 8px 0;
            padding-left: 25px;
            position: relative;
        }
        
        .defense-card li:before {
            content: "✓";
            position: absolute;
            left: 0;
            color: #4ade80;
            font-weight: bold;
            font-size: 1.2em;
        }
        
        .timeline {
            position: relative;
            padding-left: 50px;
            margin: 30px 0;
        }
        
        .timeline:before {
            content: "";
            position: absolute;
            left: 20px;
            top: 0;
            bottom: 0;
            width: 3px;
            background: #667eea;
        }
        
        .timeline-item {
            position: relative;
            margin-bottom: 30px;
        }
        
        .timeline-item:before {
            content: "";
            position: absolute;
            left: -38px;
            top: 0;
            width: 20px;
            height: 20px;
            border-radius: 50%;
            background: #667eea;
            border: 3px solid white;
            box-shadow: 0 0 0 3px #667eea;
        }
        
        .comparison-table {
            width: 100%;
            border-collapse: collapse;
            margin: 20px 0;
        }
        
        .comparison-table th,
        .comparison-table td {
            padding: 15px;
            text-align: left;
            border-bottom: 1px solid #ddd;
        }
        
        .comparison-table th {
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            color: white;
            font-weight: bold;
        }
        
        .comparison-table tr:hover {
            background: #f5f5f5;
        }
        
        .emoji {
            font-size: 1.5em;
            margin-right: 10px;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>🔐 KERBEROASTING - Schémas Détaillés</h1>
        
        <div class="tabs">
            <button class="tab-button active" onclick="showTab('attack')">
                🎯 L'Attaque
            </button>
            <button class="tab-button" onclick="showTab('defense')">
                🛡️ Les Défenses
            </button>
        </div>
        
        <!-- SCHÉMA 1 : L'ATTAQUE -->
        <div id="attack" class="schema-container active">
            <h2 class="schema-title">🎯 Attaque Kerberoasting - Flux Détaillé</h2>
            
            <!-- ÉTAPE 0 : CONTEXTE -->
            <div class="step">
                <div class="step-header">
                    <div class="step-number">0</div>
                    <span>CONTEXTE INITIAL</span>
                </div>
                
                <div class="flow-diagram">
                    <div class="actor attacker">
                        <div class="actor-icon">🥷</div>
                        <div class="actor-title">Attaquant (Jean)</div>
                        <div>Compte standard compromis</div>
                        <div>Aucun privilège admin</div>
                    </div>
                    
                    <div class="arrow">→</div>
                    
                    <div class="actor ad">
                        <div class="actor-icon">🗄️</div>
                        <div class="actor-title">Active Directory</div>
                        <div>Domaine: CORP.LOCAL</div>
                        <div>Comptes de service avec SPN</div>
                    </div>
                </div>
                
                <div class="info-box">
                    <strong>💡 Point de départ :</strong> L'attaquant a déjà compromis un compte utilisateur basique (phishing, password spray, etc.). Il cherche maintenant à élever ses privilèges.
                </div>
            </div>
            
            <!-- ÉTAPE 1 : ÉNUMÉRATION -->
            <div class="step">
                <div class="step-header">
                    <div class="step-number">1</div>
                    <span>ÉNUMÉRATION DES SPN</span>
                </div>
                
                <div class="flow-diagram">
                    <div class="actor attacker">
                        <div class="actor-icon">🔍</div>
                        <div class="actor-title">Attaquant</div>
                        <div>"Quels comptes ont des SPN ?"</div>
                    </div>
                    
                    <div class="arrow">→</div>
                    
                    <div class="actor ad">
                        <div class="actor-icon">📋</div>
                        <div class="actor-title">Active Directory</div>
                        <div>Retourne la liste des comptes</div>
                    </div>
                </div>
                
                <div class="code-box">
# PowerShell - Énumération des SPN
Get-ADUser -Filter {ServicePrincipalName -ne "$null"} `
    -Properties ServicePrincipalName | 
    Select Name, ServicePrincipalName
                </div>
                
                <div class="message-box">
                    <strong>📊 Résultat de l'énumération :</strong><br><br>
                    ✓ svc_sql_prod → MSSQLSvc/srv-sql01.corp.local:1433<br>
                    ✓ svc_exchange → exchangeMDB/mail.corp.local<br>
                    ✓ svc_backup_admin → cifs/backup-srv.corp.local<br>
                    ✓ svc_sharepoint → HTTP/sharepoint.corp.local
                </div>
                
                <div class="danger-box">
                    <strong>⚠️ Cibles prioritaires :</strong> Les comptes avec des noms comme "admin", "backup", "migration" sont souvent membres de groupes privilégiés (Domain Admins, Backup Operators...).
                </div>
            </div>
            
            <!-- ÉTAPE 2 : DEMANDE DE TGS -->
            <div class="step">
                <div class="step-header">
                    <div class="step-number">2</div>
                    <span>DEMANDE DE TGS (TICKET GRANTING SERVICE)</span>
                </div>
                
                <div class="timeline">
                    <div class="timeline-item">
                        <div class="message-box">
                            <strong>Attaquant → KDC :</strong><br>
                            "Je veux accéder au service MSSQLSvc/srv-sql01.corp.local:1433"<br>
                            <em>(Avec mon TGT valide comme preuve d'identité)</em>
                        </div>
                    </div>
                    
                    <div class="timeline-item">
                        <div class="message-box">
                            <strong>KDC vérifie :</strong><br>
                            ✓ TGT valide ? OUI<br>
                            ✓ Utilisateur authentifié ? OUI<br>
                            ✓ Service existe ? OUI<br>
                            → <span class="highlight">Aucune vérification si l'utilisateur va VRAIMENT utiliser le service !</span>
                        </div>
                    </div>
                    
                    <div class="timeline-item">
                        <div class="success-box">
                            <strong>KDC → Attaquant :</strong><br>
                            "Voici ton TGS pour accéder à SQL Server"<br><br>
                            <strong>🔑 CRUCIAL :</strong> Le TGS est chiffré avec le <span class="highlight">Hash NTLM du password du compte svc_sql_prod</span>
                        </div>
                    </div>
                </div>
                
                <div class="code-box">
# Avec Rubeus (outil de pentest)
.\Rubeus.exe kerberoast /outfile:hashes.txt

# Avec Impacket (Python)
GetUserSPNs.py CORP.LOCAL/jean:password -dc-ip 192.168.1.10 -request
                </div>
                
                <div class="info-box">
                    <strong>✅ Opération 100% légitime :</strong><br>
                    • Aucune tentative de connexion au serveur SQL<br>
                    • Aucun échec d'authentification<br>
                    • Aucune alerte générée par défaut<br>
                    • Kerberos fonctionne comme prévu
                </div>
            </div>
            
            <!-- ÉTAPE 3 : EXTRACTION DU HASH -->
            <div class="step">
                <div class="step-header">
                    <div class="step-number">3</div>
                    <span>EXTRACTION DU HASH DEPUIS LE TGS</span>
                </div>
                
                <div class="flow-diagram">
                    <div class="actor">
                        <div class="actor-icon">🎫</div>
                        <div class="actor-title">TGS Reçu</div>
                        <div style="font-size: 0.9em;">Ticket chiffré</div>
                    </div>
                    
                    <div class="arrow">→</div>
                    
                    <div class="actor attacker">
                        <div class="actor-icon">🔧</div>
                        <div class="actor-title">Extraction</div>
                        <div>Rubeus / Impacket</div>
                    </div>
                    
                    <div class="arrow">→</div>
                    
                    <div class="actor">
                        <div class="actor-icon">📝</div>
                        <div class="actor-title">Hash Extrait</div>
                        <div style="font-size: 0.9em;">Format Hashcat</div>
                    </div>
                </div>
                
                <div class="code-box">
# Format du hash extrait (type Kerberos 5 TGS-REP)
$krb5tgs$23$*svc_sql_prod$CORP.LOCAL$MSSQLSvc/srv-sql01.corp.local:1433*$
a1b2c3d4e5f6789abcdef0123456789abcdef0123456789abcdef0123456789
[...données chiffrées avec le hash NTLM du password...]
xyz9876543210fedcba9876543210fedcba9876543210fedcba9876543210
                </div>
                
                <div class="danger-box">
                    <strong>🎯 Ce que contient ce hash :</strong><br><br>
                    <strong>Partie publique :</strong><br>
                    • Nom du compte : svc_sql_prod<br>
                    • Domaine : CORP.LOCAL<br>
                    • SPN demandé : MSSQLSvc/srv-sql01.corp.local:1433<br><br>
                    <strong>Partie chiffrée (la cible) :</strong><br>
                    • Données chiffrées avec le Hash NTLM du password de svc_sql_prod<br>
                    • C'est cette partie qu'on va casser !
                </div>
            </div>
            
            <!-- ÉTAPE 4 : CASSAGE OFFLINE -->
            <div class="step">
                <div class="step-header">
                    <div class="step-number">4</div>
                    <span>CASSAGE OFFLINE DU HASH (BRUTE FORCE)</span>
                </div>
                
                <div class="info-box">
                    <strong>🏠 IMPORTANT :</strong> Cette étape se passe sur la machine de l'attaquant, complètement OFFLINE. Aucune connexion au réseau de l'entreprise !
                </div>
                
                <div class="code-box">
# Hashcat - Cassage du hash TGS
hashcat -m 13100 hashes.txt /usr/share/wordlists/rockyou.txt --force

# -m 13100 = Mode Kerberos 5 TGS-REP etype 23 (RC4)
# rockyou.txt = Dictionnaire de 14 millions de passwords
                </div>
                
                <div class="timeline">
                    <div class="timeline-item">
                        <div class="message-box">
                            <strong>Test #1 :</strong> "Password123"<br>
                            → Calcul du Hash NTLM de "Password123"<br>
                            → Chiffrement d'une partie du TGS avec ce hash<br>
                            → Comparaison avec le TGS réel<br>
                            → ❌ Pas de correspondance
                        </div>
                    </div>
                    
                    <div class="timeline-item">
                        <div class="message-box">
                            <strong>Test #2 :</strong> "Welcome2024"<br>
                            → Calcul du Hash NTLM de "Welcome2024"<br>
                            → Chiffrement d'une partie du TGS avec ce hash<br>
                            → Comparaison avec le TGS réel<br>
                            → ❌ Pas de correspondance
                        </div>
                    </div>
                    
                    <div class="timeline-item">
                        <div class="message-box">
                            <strong>Test #458,392 :</strong> "SqlService2019!"<br>
                            → Calcul du Hash NTLM de "SqlService2019!"<br>
                            → Chiffrement d'une partie du TGS avec ce hash<br>
                            → Comparaison avec le TGS réel<br>
                            → ✅ <span class="highlight">CORRESPONDANCE PARFAITE !</span>
                        </div>
                    </div>
                </div>
                
                <div class="success-box">
                    <strong>🎉 PASSWORD CRAQUÉ !</strong><br><br>
                    Compte : svc_sql_prod<br>
                    Password : <span class="highlight">SqlService2019!</span><br><br>
                    Temps de cassage : 4 minutes 32 secondes<br>
                    (avec une RTX 4090 @ 100 milliards de hash/s)
                </div>
                
                <div class="danger-box">
                    <strong>⚠️ Pourquoi c'est si dangereux :</strong><br><br>
                    ✗ Aucune limite de tentatives (pas de lockout)<br>
                    ✗ Aucune alerte en temps réel<br>
                    ✗ Peut prendre des jours/semaines sans détection<br>
                    ✗ L'attaquant peut utiliser un cluster GPU massif
                </div>
            </div>
            
            <!-- ÉTAPE 5 : EXPLOITATION -->
            <div class="step">
                <div class="step-header">
                    <div class="step-number">5</div>
                    <span>EXPLOITATION POST-COMPROMISSION</span>
                </div>
                
                <div class="timeline">
                    <div class="timeline-item">
                        <div class="success-box">
                            <strong>✅ Authentification avec le compte compromis</strong>
                            <div class="code-box" style="margin-top: 10px;">
runas /user:CORP\svc_sql_prod cmd.exe
# Password: SqlService2019!
                            </div>
                        </div>
                    </div>
                    
                    <div class="timeline-item">
                        <div class="message-box">
                            <strong>🔍 Vérification des privilèges</strong>
                            <div class="code-box" style="margin-top: 10px;">
whoami /groups

# Résultat :
CORP\Domain Admins
CORP\Backup Operators
CORP\Schema Admins
                            </div>
                        </div>
                    </div>
                    
                    <div class="timeline-item">
                        <div class="danger-box">
                            <strong>💀 COMPROMISSION TOTALE DU DOMAINE</strong><br><br>
                            Avec Domain Admins, l'attaquant peut :<br><br>
                            🔓 Dumper tous les hashs NTLM du domaine (DCSync)<br>
                            🔓 Créer des comptes admin persistants<br>
                            🔓 Installer des backdoors sur tous les serveurs<br>
                            🔓 Exfiltrer toutes les données sensibles<br>
                            🔓 Déployer des ransomwares sur tout le réseau
                        </div>
                    </div>
                </div>
            </div>
            
            <!-- RÉCAPITULATIF -->
            <div class="step">
                <div class="step-header">
                    <div class="step-number">📊</div>
                    <span>RÉCAPITULATIF DE L'ATTAQUE</span>
                </div>
                
                <table class="comparison-table">
                    <thead>
                        <tr>
                            <th>Étape</th>
                            <th>Action</th>
                            <th>Détectable ?</th>
                            <th>Privilèges requis</th>
                        </tr>
                    </thead>
                    <tbody>
                        <tr>
                            <td><strong>1. Énumération SPN</strong></td>
                            <td>Requête LDAP dans AD</td>
                            <td>❌ Difficile (activité normale)</td>
                            <td>Utilisateur standard</td>
                        </tr>
                        <tr>
                            <td><strong>2. Demande TGS</strong></td>
                            <td>Requête Kerberos légitime</td>
                            <td>❌ Non par défaut</td>
                            <td>Utilisateur standard</td>
                        </tr>
                        <tr>
                            <td><strong>3. Extraction hash</strong></td>
                            <td>Parsing du ticket local</td>
                            <td>❌ Aucune trace réseau</td>
                            <td>Aucun</td>
                        </tr>
                        <tr>
                            <td><strong>4. Cassage offline</strong></td>
                            <td>Brute force local</td>
                            <td>❌ Complètement invisible</td>
                            <td>Aucun</td>
                        </tr>
                        <tr>
                            <td><strong>5. Exploitation</strong></td>
                            <td>Authentification légitime</td>
                            <td>⚠️ Possible (comportement anormal)</td>
                            <td>Domain Admin (si compte ciblé)</td>
                        </tr>
                    </tbody>
                </table>
            </div>
        </div>
        
        <!-- SCHÉMA 2 : LES DÉFENSES -->
        <div id="defense" class="schema-container">
            <h2 class="schema-title">🛡️ Défense Contre le Kerberoasting</h2>
            
            <!-- STRATÉGIE GLOBALE -->
            <div class="step">
                <div class="step-header">
                    <div class="step-number">🎯</div>
                    <span>STRATÉGIE DE DÉFENSE EN PROFONDEUR</span>
                </div>
                
                <div class="info-box">
                    <strong>💡 Principe clé :</strong> Il n'existe pas de solution miracle. La défense doit être multi-couches : prévention + détection + réponse.
                </div>
                
                <div class="defense-grid">
                    <div class="defense-card">
                        <h3>🔒 PRÉVENTION</h3>
                        <ul>
                            <li>Empêcher l'attaque de réussir</li>
                            <li>Rendre le cassage impossible ou extrêmement long</li>
                            <li>Réduire la surface d'attaque</li>
                        </ul>
                    </div>
                    
                    <div class="defense-card">
                        <h3>👁️ DÉTECTION</h3>
                        <ul>
                            <li>Identifier l'attaque en cours</li>
                            <li>Alertes en temps réel</li>
                            <li>Honeypots et canary tokens</li>
                        </ul>
                    </div>
                    
                    <div class="defense-card">
                        <h3>🚨 RÉPONSE</h3>
                        <ul>
                            <li>Isolation rapide de la menace</li>
                            <li>Révocation des tickets compromis</li>
                            <li>Investigation forensique</li>
                        </ul>
                    </div>
                </div>
            </div>
            
            <!-- DÉFENSE 1 : PASSWORDS FORTS -->
            <div class="step">
                <div class="step-header">
                    <div class="step-number">1</div>
                    <span>PASSWORDS COMPLEXES POUR LES COMPTES DE SERVICE</span>
                </div>
                
                <div class="success-box">
                    <strong>🎯 Objectif :</strong> Rendre le cassage du hash <span class="highlight">mathématiquement impossible</span> même avec des GPUs puissants.
                </div>
                
                <div class="timeline">
                    <div class="timeline-item">
                        <div class="message-box">
                            <strong>❌ Password FAIBLE (cassable en 5 minutes) :</strong><br>
                            <code>SqlService2019!</code><br>
                            • Longueur : 15 caractères<br>
                            • Motif : Mot courant + année + caractère spécial<br>
                            • Présent dans les dictionnaires
                        </div>
                    </div>
                    
                    <div class="timeline-item">
                        <div class="success-box">
                            <strong>✅ Password FORT (cassage : plusieurs millénaires) :</strong><br>
                            <code>X9$mK#pL2@vN8!qR4&wT6^yU</code><br>
                            • Longueur : 25+ caractères<br>
                            • Aléatoire complet<br>
                            • Majuscules + minuscules + chiffres + symboles<br>
                            • Temps de cassage estimé : <span class="highlight">6,7 milliards d'années</span>
                        </div>
                    </div>
                </div>
                
                <table class="comparison-table">
                    <thead>
                        <tr>
                            <th>Longueur</th>
                            <th>Complexité</th>
                            <th>Temps de cassage (RTX 4090)</th>
                            <th>Recommandation</th>
                        </tr>
                    </thead>
                    <tbody>
                        <tr>
                            <td>8 caractères</td>
                            <td>Alphanumérique</td>
                            <td>2 secondes</td>
                            <td>❌ Dangereux</td>
                        </tr>
                        <tr>
                            <td>12 caractères</td>
                            <td>+ Symboles</td>
                            <td>3 minutes</td>
                            <td>❌ Insuffisant</td>
                        </tr>
                        <tr>
                            <td>15 caractères</td>
                            <td>+ Symboles</td>
                            <td>2 jours</td>
                            <td>⚠️ Faible</td>
                        </tr>
                        <tr>
                            <td>20 caractères</td>
                            <td>+ Symboles</td>
                            <td>850 ans</td>
                            <td>✅ Acceptable</td>
                        </tr>
                        <tr>
                            <td>25+ caractères</td>
                            <td>+ Symboles</td>
                            <td>Milliards d'années</td>
                            <td>✅ Recommandé</td>
                        </tr>
                    </tbody>
                </table>
                
                <div class="code-box">
# PowerShell - Génération d'un password fort (25 caractères)
-join ((48..57) + (65..90) + (97..122) + (33,35,36,37,38,42,43,45,46,47,58,61,63,64,94) | 
    Get-Random -Count 25 | 
    ForEach-Object {[char]$_})

# Exemple de résultat : K#9mP@7vL2!qN8$wR4&tY6^
                </div>
            </div>
            
            <!-- DÉFENSE 2 : gMSA -->
            <div class="step">
                <div class="step-header">
                    <div class="step-number">2</div>
                    <span>DÉPLOIEMENT DE gMSA (GROUP MANAGED SERVICE ACCOUNTS)</span>
                </div>
                
                <div class="success-box">
                    <strong>🎯 Solution ultime :</strong> Les gMSA sont des comptes de service dont le password est <span class="highlight">généré et géré automatiquement par Active Directory</span>.
                </div>
                
                <div class="defense-grid">
                    <div class="defense-card">
                        <h3>✨ Avantages gMSA</h3>
                        <ul>
                            <li>Password de 120 caractères (impossible à casser)</li>
                            <li>Rotation automatique tous les 30 jours</li>
                            <li>Aucun humain ne connaît le password</li>
                            <li>Pas de gestion manuelle</li>
                            <li>Immunité totale contre Kerberoasting</li>
                        </ul>
                    </div>
                    
                    <div class="defense-card" style="background: linear-gradient(135deg, #fa709a 0%, #fee140 100%);">
                        <h3>⚠️ Comptes Standards</h3>
                        <ul>
                            <li style="color: #333;">Password manuel (souvent faible)</li>
                            <li style="color: #333;">Rotation rare ou jamais</li>
                            <li style="color: #333;">Stocké dans des docs/scripts</li>
                            <li style="color: #333;">Risque de fuite</li>
                            <li style="color: #333;">Vulnérable au Kerberoasting</li>
                        </ul>
                    </div>
                </div>
                
                <div class="code-box">
# PowerShell - Création d'un gMSA pour SQL Server

# 1. Créer la KDS Root Key (une fois par domaine)
Add-KdsRootKey -EffectiveTime ((Get-Date).AddHours(-10))

# 2. Créer le gMSA
New-ADServiceAccount -Name "gMSA-SQL-Prod" `
    -DNSHostName "srv-sql01.corp.local" `
    -PrincipalsAllowedToRetrieveManagedPassword "SRV-SQL01$"

# 3. Installer sur le serveur SQL
Install-ADServiceAccount -Identity "gMSA-SQL-Prod"

# 4. Configurer le service SQL pour utiliser le gMSA
# Compte : CORP\gMSA-SQL-Prod$
# Password : (laissez vide - géré automatiquement)
                </div>
                
                <div class="info-box">
                    <strong>💡 Résultat :</strong> Le compte gMSA-SQL-Prod$ a un password de 120 caractères aléatoires qui change automatiquement tous les 30 jours. Temps de cassage : <span class="highlight">Plusieurs fois l'âge de l'univers</span>.
                </div>
            </div>
            
            <!-- DÉFENSE 3 : AES ENCRYPTION -->
            <div class="step">
                <div class="step-header">
                    <div class="step-number">3</div>
                    <span>FORCER LE CHIFFREMENT AES (DÉSACTIVER RC4)</span>
                </div>
                
                <div class="danger-box">
                    <strong>⚠️ Problème RC4 :</strong> L'algorithme RC4 utilisé dans Kerberos (type 23) est obsolète et plus rapide à casser que AES.
                </div>
                
                <table class="comparison-table">
                    <thead>
                        <tr>
                            <th>Algorithme</th>
                            <th>Type Kerberos</th>
                            <th>Vitesse de cassage</th>
                            <th>Recommandation</th>
                        </tr>
                    </thead>
                    <tbody>
                        <tr>
                            <td>RC4-HMAC</td>
                            <td>Type 23</td>
                            <td>100 milliards hash/s</td>
                            <td>❌ À DÉSACTIVER</td>
                        </tr>
                        <tr>
                            <td>AES128-CTS</td>
                            <td>Type 17</td>
                            <td>10 millions hash/s</td>
                            <td>✅ Acceptable</td>
                        </tr>
                        <tr>
                            <td>AES256-CTS</td>
                            <td>Type 18</td>
                            <td>5 millions hash/s</td>
                            <td>✅ Recommandé</td>
                        </tr>
                    </tbody>
                </table>
                
                <div class="success-box">
                    <strong>✅ Gain :</strong> Forcer AES256 ralentit le cassage d'un facteur <span class="highlight">20 000x</span> !
                </div>
                
                <div class="code-box">
# GPO - Désactivation de RC4 dans tout le domaine

# Chemin GPO :
Computer Configuration → Windows Settings → Security Settings → 
Local Policies → Security Options

# Paramètre :
"Network security: Configure encryption types allowed for Kerberos"

# Configuration :
☑ AES128_HMAC_SHA1
☑ AES256_HMAC_SHA1
☐ RC4_HMAC_MD5      ← DÉSACTIVER
☐ DES_CBC_CRC       ← DÉSACTIVER
☐ DES_CBC_MD5       ← DÉSACTIVER
                </div>
            </div>
            
            <!-- DÉFENSE 4 : PRINCIPE DU MOINDRE PRIVILÈGE -->
            <div class="step">
                <div class="step-header">
                    <div class="step-number">4</div>
                    <span>PRINCIPE DU MOINDRE PRIVILÈGE</span>
                </div>
                
                <div class="danger-box">
                    <strong>❌ Erreur critique fréquente :</strong><br>
                    Les comptes de service sont membres de <strong>Domain Admins</strong> "pour simplifier les permissions".
                </div>
                
                <div class="timeline">
                    <div class="timeline-item">
                        <div class="message-box">
                            <strong>❌ Mauvaise pratique :</strong><br>
                            <code>svc_sql_prod</code> → Membre de <strong>Domain Admins</strong><br><br>
                            Conséquence : Si cassé → Compromission TOTALE du domaine
                        </div>
                    </div>
                    
                    <div class="timeline-item">
                        <div class="success-box">
                            <strong>✅ Bonne pratique :</strong><br>
                            <code>svc_sql_prod</code> → Permissions minimales :<br>
                            • Lecture/écriture sur les bases de données SQL uniquement<br>
                            • Aucun privilège admin sur le domaine<br>
                            • Aucun accès aux autres serveurs<br><br>
                            Conséquence : Si cassé → Impact limité au service SQL
                        </div>
                    </div>
                </div>
                
                <div class="info-box">
                    <strong>💡 Règle d'or :</strong> Un compte de service ne doit JAMAIS être Domain Admin. Utilisez des groupes dédiés avec permissions granulaires.
                </div>
            </div>
            
            <!-- DÉFENSE 5 : DÉTECTION PAR EVENT LOGS -->
            <div class="step">
                <div class="step-header">
                    <div class="step-number">5</div>
                    <span>DÉTECTION VIA EVENT LOGS (SIEM)</span>
                </div>
                
                <div class="success-box">
                    <strong>🎯 Event ID 4769 :</strong> Chaque demande de TGS génère cet événement sur le contrôleur de domaine.
                </div>
                
                <div class="defense-grid">
                    <div class="defense-card">
                        <h3>🚨 Patterns suspects</h3>
                        <ul>
                            <li>Volume anormal : +10 TGS en 5 minutes</li>
                            <li>SPN rarement utilisés</li>
                            <li>Encryption Type = RC4 (0x17)</li>
                            <li>Requêtes hors heures ouvrables</li>
                            <li>Comptes qui ne demandent jamais de TGS</li>
                        </ul>
                    </div>
                    
                    <div class="defense-card">
                        <h3>✅ Activité légitime</h3>
                        <ul>
                            <li>TGS pour krbtgt</li>
                            <li>TGS pour comptes machines ($)</li>
                            <li>Volume normal (1-2 par session)</li>
                            <li>Services utilisés régulièrement</li>
                            <li>Heures ouvrables</li>
                        </ul>
                    </div>
                </div>
                
                <div class="code-box">
# Splunk Query - Détection Kerberoasting

index=windows EventCode=4769 
| where TicketEncryptionType="0x17"  # RC4
| where ServiceName!="krbtgt" AND ServiceName!="*$"
| stats count by src_ip, user, ServiceName
| where count > 5
| table _time, user, src_ip, ServiceName, count

# Alerte si count > 10 en 5 minutes
                </div>
                
                <div class="code-box">
# Azure Sentinel (KQL) - Détection avancée

SecurityEvent
| where EventID == 4769
| where ServiceName !endswith "$" and ServiceName != "krbtgt"
| where TicketEncryptionType == "0x17"  // RC4
| summarize 
    RequestCount = count(),
    UniqueServices = dcount(ServiceName),
    Services = make_set(ServiceName)
    by Account, IpAddress, bin(TimeGenerated, 5m)
| where RequestCount > 5 OR UniqueServices > 3
| project TimeGenerated, Account, IpAddress, RequestCount, UniqueServices, Services
                </code>
            </div>
            </div>
            
            <!-- DÉFENSE 6 : HONEYPOT SPN -->
            <div class="step">
                <div class="step-header">
                    <div class="step-number">6</div>
                    <span>HONEYPOT SPN (CANARY TOKENS)</span>
                </div>
                
                <div class="success-box">
                    <strong>🎯 Stratégie :</strong> Créer des comptes de service "leurres" avec des noms attractifs. <span class="highlight">Toute demande de TGS = Attaque confirmée à 100%</span>.
                </div>
                
                <div class="code-box">
# PowerShell - Création d'un compte honeypot

# 1. Créer le compte avec un nom attractif
New-ADUser -Name "SQL_Backup_Admin" `
    -AccountPassword (ConvertTo-SecureString "Tr3sC0mpl3x3P@ssw0rd!2024#Sec" -AsPlainText -Force) `
    -Enabled $true `
    -Description "HONEYPOT - Ne pas utiliser"

# 2. Attribuer un SPN
setspn -A MSSQLSvc/honeypot-backup.corp.local:1433 SQL_Backup_Admin

# 3. Configurer une alerte SIEM
# Event 4769 WHERE ServiceName = "MSSQLSvc/honeypot-backup.corp.local:1433"
# → Alerte critique immédiate
                </div>
                
                <div class="info-box">
                    <strong>💡 Noms attractifs pour honeypots :</strong><br>
                    • <code>SQL_Admin_Backup</code><br>
                    • <code>Exchange_Migration_Svc</code><br>
                    • <code>VMware_vCenter_Admin</code><br>
                    • <code>Backup_Domain_Admin</code><br>
                    • <code>SAP_Super_User</code>
                </div>
                
                <div class="danger-box">
                    <strong>⚠️ CRITIQUE :</strong> Ce compte ne doit JAMAIS être utilisé par aucun service. Toute activité = Compromission en cours !
                </div>
            </div>
            
            <!-- DÉFENSE 7 : ROTATION DES PASSWORDS -->
            <div class="step">
                <div class="step-header">
                    <div class="step-number">7</div>
                    <span>ROTATION RÉGULIÈRE DES PASSWORDS</span>
                </div>
                
                <div class="timeline">
                    <div class="timeline-item">
                        <div class="danger-box">
                            <strong>❌ Situation actuelle (trop fréquente) :</strong><br>
                            Password défini en 2015 → Jamais changé depuis 10 ans
                        </div>
                    </div>
                    
                    <div class="timeline-item">
                        <div class="success-box">
                            <strong>✅ Bonne pratique :</strong><br>
                            • Rotation tous les 90 jours (minimum)<br>
                            • Rotation tous les 30 jours (recommandé)<br>
                            • Automatisée via scripts ou gMSA
                        </div>
                    </div>
                </div>
                
                <div class="info-box">
                    <strong>💡 Avantage :</strong> Même si un hash est cassé, le password ne sera valide que pour une période limitée.
                </div>
            </div>
            
            <!-- DÉFENSE 8 : AUDIT ET MONITORING -->
            <div class="step">
                <div class="step-header">
                    <div class="step-number">8</div>
                    <span>AUDIT CONTINU DES COMPTES DE SERVICE</span>
                </div>
                
                <div class="code-box">
# PowerShell - Script d'audit des comptes de service

# Lister tous les comptes avec SPN
$ServiceAccounts = Get-ADUser -Filter {ServicePrincipalName -ne "$null"} `
    -Properties ServicePrincipalName, MemberOf, PasswordLastSet, Enabled

foreach ($account in $ServiceAccounts) {
    $groups = $account.MemberOf | ForEach-Object {
        (Get-ADGroup $_).Name
    }
    
    # Vérifications de sécurité
    $isDomainAdmin = $groups -contains "Domain Admins"
    $passwordAge = (Get-Date) - $account.PasswordLastSet
    
    [PSCustomObject]@{
        Account = $account.Name
        SPN = $account.ServicePrincipalName -join "; "
        IsDomainAdmin = $isDomainAdmin
        PasswordAgeDays = $passwordAge.Days
        Enabled = $account.Enabled
        Alert = ($isDomainAdmin -or $passwordAge.Days -gt 90)
    }
}
                </div>
                
                <div class="success-box">
                    <strong>✅ Automatisation :</strong> Planifier ce script toutes les semaines et envoyer un rapport aux équipes de sécurité.
                </div>
            </div>
            
            <!-- PLAYBOOK DE RÉPONSE -->
            <div class="step">
                <div class="step-header">
                    <div class="step-number">🚨</div>
                    <span>PLAYBOOK DE RÉPONSE À INCIDENT</span>
                </div>
                
                <div class="timeline">
                    <div class="timeline-item">
                        <div class="danger-box">
                            <strong>⚠️ ALERTE DÉTECTÉE : Kerberoasting en cours</strong>
                        </div>
                    </div>
                    
                    <div class="timeline-item">
                        <div class="message-box">
                            <strong>🔒 ÉTAPE 1 : ISOLATION IMMÉDIATE (< 5 minutes)</strong><br>
                            • Bloquer l'IP source au firewall<br>
                            • Désactiver le compte utilisateur compromis<br>
                            • Isoler la machine de l'attaquant (VLAN quarantaine)
                        </div>
                    </div>
                    
                    <div class="timeline-item">
                        <div class="message-box">
                            <strong>🔑 ÉTAPE 2 : RÉVOCATION ET ROTATION (< 15 minutes)</strong><br>
                            • Reset des passwords de TOUS les comptes SPN<br>
                            • Purge des tickets Kerberos actifs<br>
                            • Révocation des sessions authentifiées
                        </div>
                    </div>
                    
                    <div class="timeline-item">
                        <div class="message-box">
                            <strong>🔍 ÉTAPE 3 : INVESTIGATION (< 1 heure)</strong><br>
                            • Memory dump de la machine attaquante<br>
                            • Analyse des Event Logs sur 30 jours<br>
                            • Recherche de reconnaissance préalable (LDAP queries)<br>
                            • Vérification des autres comptes de l'attaquant
                        </div>
                    </div>
                    
                    <div class="timeline-item">
                        <div class="success-box">
                            <strong>🛡️ ÉTAPE 4 : HARDENING POST-INCIDENT</strong><br>
                            • Déploiement de gMSA sur tous les services<br>
                            • Désactivation globale de RC4<br>
                            • Implémentation de honeypots SPN<br>
                            • Renforcement du monitoring SIEM<br>
                            • Formation de sensibilisation des équipes
                        </div>
                    </div>
                </div>
            </div>
            
            <!-- CHECKLIST FINALE -->
            <div class="step">
                <div class="step-header">
                    <div class="step-number">✅</div>
                    <span>CHECKLIST DE SÉCURITÉ KERBEROS</span>
                </div>
                
                <table class="comparison-table">
                    <thead>
                        <tr>
                            <th>Mesure de Sécurité</th>
                            <th>Priorité</th>
                            <th>Difficulté</th>
                            <th>Impact</th>
                        </tr>
                    </thead>
                    <tbody>
                        <tr>
                            <td>✅ Audit des comptes SPN existants</td>
                            <td>🔴 Critique</td>
                            <td>Facile</td>
                            <td>Visibilité</td>
                        </tr>
                        <tr>
                            <td>✅ Passwords 25+ caractères pour SPN</td>
                            <td>🔴 Critique</td>
                            <td>Facile</td>
                            <td>Prévention</td>
                        </tr>
                        <tr>
                            <td>✅ Déploiement de gMSA</td>
                            <td>🔴 Critique</td>
                            <td>Moyen</td>
                            <td>Protection complète</td>
                        </tr>
                        <tr>
                            <td>✅ Désactivation de RC4 (forcer AES)</td>
                            <td>🔴 Critique</td>
                            <td>Moyen</td>
                            <td>Ralentit le cassage</td>
                        </tr>
                        <tr>
                            <td>✅ Retrait des SPN de Domain Admins</td>
                            <td>🔴 Critique</td>
                            <td>Facile</td>
                            <td>Réduction impact</td>
                        </tr>
                        <tr>
                            <td>✅ Monitoring Event ID 4769</td>
                            <td>🟠 Élevée</td>
                            <td>Moyen</td>
                            <td>Détection</td>
                        </tr>
                        <tr>
                            <td>✅ Honeypot SPN</td>
                            <td>🟠 Élevée</td>
                            <td>Facile</td>
                            <td>Alerte précoce</td>
                        </tr>
                        <tr>
                            <td>✅ Rotation passwords (90 jours)</td>
                            <td>🟡 Moyenne</td>
                            <td>Moyen</td>
                            <td>Limitation temporelle</td>
                        </tr>
                        <tr>
                            <td>✅ Playbook de réponse documenté</td>
                            <td>🟠 Élevée</td>
                            <td>Facile</td>
                            <td>Réaction rapide</td>
                        </tr>
                    </tbody>
                </table>
            </div>
        </div>
    </div>
    
    <script>
        function showTab(tabName) {
            // Masquer tous les schémas
            document.querySelectorAll('.schema-container').forEach(container => {
                container.classList.remove('active');
            });
            
            // Désactiver tous les boutons
            document.querySelectorAll('.tab-button').forEach(button => {
                button.classList.remove('active');
            });
            
            // Afficher le schéma sélectionné
            document.getElementById(tabName).classList.add('active');
            
            // Activer le bouton correspondant
            event.target.classList.add('active');
        }
    </script>
</body>
</html># Im4d21.github.io
Interactive page that explains kerberoasting and provides defense/detection recommendations.
