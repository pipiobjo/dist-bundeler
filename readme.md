

# Development Information: 

## Commands: 

./gradlew uploadPlainLiferaySP
Task wird nur gebraucht wenn Liferay-Bundle im Nexus nicht existiert.
l�dt Liferay im Nexus hoch. Die pom Version, groupId und artifactId werden �ber die gradle Properties liferayTomcatBundleVersion, liferayTomcatGroupId und liferayTomcatArtifactId konfiguriert.
Der Task uploadPlainLiferaySP hat als Dependency die Konfiguration liferayUploadZip. Die Konfiguration liferayUploadZip ist ein Referenz auf das Liferay-File auf dem lokalen Filesystem. Daher muss der komplette Name von dem Liferay-Bundle 
im gradle Property ext.liferayBundleNameLocal konfiguriert werden.


./gradlew uploadTomcatBundle
Task wird nur gebraucht wenn TOmcat-Bundle im Nexus nicht existiert.
l�dt TOmcat im Nexus hoch. Die pom Version, groupId und artifactId werden �ber die gradle Properties tomcatBundleVersion, tomcatGroupId und tomcatArtifactId konfiguriert.
Der Task uploadTomcatBundle hat als Dependency die Konfiguration tomcatUploadZip. Die Konfiguration tomcatUploadZip ist ein Referenz auf das Tomcat-File auf dem lokalen Filesystem. Daher muss der komplette Name von dem Tomcat-Bundle 
im gradle Property ext.tomcatBundleNameLocal konfiguriert werden.


./gradlew installLiferaypatchingToolExecuteInstallUpgradeAndStartTomcat
Der Task installLiferaypatchingToolExecuteInstallUpgradeAndStartTomcat l�dt liferay runter, entzippt Liferay, aktualisert patching tool, instaliert Fixpacks, aktualisiert Tomcat und aktualisert die LPKGS. Dazu ruft er folgende Tasks auf:
1- :cleanBuildTempDir
	Task l�scht den Temp-Ordner im Projekt-Ordner
2- :explodeLiferayBundle
	Task macht folgendes:
	* l�dt Liferay Bundle Zip runter 
	* Entpackt es nach temp/liferay-temp
	* Kopiert temp/liferay-temp/liferay*/ nach liferay. Das wird gemacht damit wir kein Liferay Parent-Ordner mit einem spezifischen Namen.
3- :patchingToolUpdate
	Task aktualisiert das patching-tool. Es wird gepr�ft ob im Ordner uploads\patching-tool eine neue Version vom patching-tool existiert und das 	alte patching tool im Liferay aktualisiert. 
	Im Ordner uploads\patching-tool darf maximal eine Version vom patching-tool existieren. Wenn der Ordner uploads/patching-tool leer ist dann wird 	patching-tool nicht geupdated.
4- :copyFixPacksToLiferay
	Task kopiert alle Fixpacks vom Ordner uploads/liferay-fix-packs nach temp/liferay/patching-tool/patches
5- :patchingToolAutoDiscovery
	Task schreibt die Datei default.properties in temp/liferay/patching-tool neue. In dieser Datei werden die Pfade eingestellt, die f�r das 	Ausf�hren von patchinig-tool.bat n�tig sind
6- :patchingToolExecuteInstall
	Task installiert die im Ordner temp/liferay/patching-tool/patches existierende Fixpacks im Liferay. 
7- :backupAllLiferayTomcatFiles
	Task ruft folgende Tasks auf:
	* :backupCatalinaProperties
		Task macht eine Kopie von der Datei catalina.properties im temp/liferay/tomcat*/conf und speichert es im temp/backup: Diese Datei wird nach 		dem Upgrade vom Tomcat gebraucht, weil die vom Liferay angepasst wurde und somit nicht gleich wie vom Tomcat geliefert wird. 
	* :backupExtFolder
		Task macht eine Kopie vom temp/liferay/tomcat*/lib/ext folder und speichert es im temp. Die Libraries im ext Folder kommen vom Liferay
	* :backupROOTFolder
		Task macht eine Kopie vom ROOT folder und speichert es im temp
8- :deleteOldTomcatInLiferay
	Task l�scht den Tomcat-Ordner aus Liferay. Das wird gebraucht da wir danach eine andere Version vom Tocat installieren.
9- :explodeTomcatBundle
	Task l�dt tomcat bundle Zip runter and entpackt es nach temp/liferay/tomcat
10- :deleteWebappsFolderVomLiferayTomcat
	Task l�scht den Webapp-Ordner im neuen Tomcat
11- :copyAllLiferayFilesToTomcat
	Task kopiert die Dateien nach Tomcat, die im Task :backupAllLiferayTomcatFiles gesichert wurden
12- :deleteUnwantedLpkg
	Task l�scht die nicht ben�tigte LPKS (Liferay Documentum Connector.lpkg - Liferay IP Geocoder.lpkg - Liferay Sharepoint Connector.lpkg - Liferay Sync Connector.lpkg) aus dem Ordner temp/liferay/osgi/marketplace
13- :updateProfileForPatchingTool
	Task schreibt die Datei default.properties in temp/liferay/patching-tool neue. Dies wird aufgerufen da Tomcat aktualisert wurde und somit die 	Pfade zu ROOT-Ordner und ext-Ordner ge�ndert wurden.
14- :upgradeLPKGS
	Task aktualisert die Liferay-LPKGS. Die neue LPKGS m�ssen im Ordner uploads/lpkg/marketplace existieren 
15- :startUpgradedTomcat
	Task startet das aktualisierte Tomcat nach dem Upgrade
	Nachdem Tomcat gestartet wurde muss man die Lizenz-Datei nach temp/liferay/deploy kopieren und Smoke-Test im Portal machen.


./gradlew uploadArchives
Der Task uploadArchives baut eine Liferay-dist-Bundle zusammen und l�dt es im Nexus hoch. Dazu werden folgende Task aufgerufen
1- :clean 
	Task bereinige den build-Ordner
2- :shutdownTomcat
	Task f�hrt Tomcat vom temp/liferay/tomcat*/bin runter. Dies ist n�ctig um den LiferayTomcatDist sauber zu erstellen
3- :extendLiferayDist
	Task l�scht die nicht ben�tigte Dateien f�r das Zusammenbauen von dem Dist
4- :assembleDist
	Task kommt von dem distribution-Plugin und ruft intern den anderen Task :distZip auf. Der Task :distZip baut das dist-Bundle zusammen und 	kopiert es nach build/distributions/
5- :uploadArchives
	Task l�dt das distribution zip file im nexus hoch. Die pom.version, pom.groupId und pom.artifactId werden im gradle.properties �ber die 	properties projectversion, projectgroup und projectname konfiguriert.