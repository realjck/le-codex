# Mise en ligne avec WinSCP


Il est souvent utile de pousser du contenu local vers un serveur distant en SFTP. Pour automatiser cette tâche nous pouvons utiliser le logiciel en ligne de commande [WinSCP](https://winscp.net/eng/downloads.php) (Il est préférable de choisir la version portable).

Le script PowerShell ci-dessous effectue une copie du dossier local `./site` vers  un serveur dans `/var/www/<site>` , il efface au préalable le contenu de ce-dernier avant copie :

```PowerShell
# Chemin vers l'exécutable WinSCP.com
$WinSCPPath = "C:\tools\WinSCP\WinSCP.com"

# Informations de connexion
$SftpHost = "12.34.56.178"
$SftpPort = 22
$SftpUser = "<your_user>"
$SftpPassword = "**********"
$RemoteDir = "/var/www/<site>"
$LocalDir = ".\site"

# Crée un fichier de script temporaire pour WinSCP
$WinSCPscript = @"
open sftp://$($SftpUser):$($SftpPassword)@$($SftpHost):$($SftpPort)
cd $RemoteDir
rm *
put $($LocalDir)\* -filemask="*>"
exit
"@

# Chemin du fichier de script temporaire
$scriptPath = [System.IO.Path]::GetTempFileName()

# Écrire le script WinSCP dans le fichier temporaire
Set-Content -Path $scriptPath -Value $WinSCPscript

# Exécuter WinSCP avec le script temporaire
& $WinSCPPath /script=$scriptPath

# Supprimer le fichier de script temporaire
Remove-Item -Path $scriptPath
```