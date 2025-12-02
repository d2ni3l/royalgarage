const sql = ` WITH RankedResults AS ( SELECT *, ROW_NUMBER() OVER (PARTITION BY ID ORDER BY Source DESC) AS rn FROM ( SELECT aziende.ID, aziende.RagioneSociale AS Nome, aziende.RagioneSociale AS Descrizione, aziende.Località, NULL AS contattoID, aziende.DescrizioneBlackList AS DescrizioneBlackList, aziende.Blacklist AS Blacklist, 'Azienda' AS Source FROM aziende WHERE aziende.RagioneSociale LIKE ${searchTerm} AND aziende.deleted_at IS NULL

UNION ALL

SELECT aziende.ID AS ID, CONCAT(contatti.Nome, ' ', contatti.Cognome) AS Nome, contatti.Nome AS Descrizione, aziende.Località, contatti.ID AS contattoID, aziende.DescrizioneBlackList AS DescrizioneBlackList, aziende.Blacklist AS Blacklist, 'Azienda' AS Source FROM contatti INNER JOIN aziende ON aziende.ID = contatti.IDAzienda WHERE (contatti.Nome LIKE ${searchTerm} OR contatti.Cognome LIKE ${searchTerm}) AND aziende.deleted_at IS NULL

UNION ALL

SELECT denominazioni.ID AS ID, denominazioni.Denominazione AS Nome, contatti.Nome AS Descrizione, aziende.Località, contatti.ID AS contattoID, aziende.DescrizioneBlackList AS DescrizioneBlackList, aziende.Blacklist AS Blacklist, 'Denominazione' AS Source FROM contatti INNER JOIN aziende ON aziende.ID = contatti.IDAzienda INNER JOIN denominazioni ON denominazioni.ID = contatti.IDDenominazione WHERE (denominazioni.Denominazione LIKE ${searchTerm}) AND aziende.deleted_at IS NULL ) AS combined_results ) SELECT * FROM RankedResults WHERE rn = 1 AND Nome IS NOT NULL ORDER BY TRIM(Nome) LIMIT 40;`;
