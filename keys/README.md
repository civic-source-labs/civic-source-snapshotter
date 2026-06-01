# Keys

public repoには暗号化用のage public recipientだけを置きます。

置いてよい:

- `owner-age-recipient.txt`

置いてはいけない:

- age identity / private key
- 復号済みartifact
- downstream DB password
- GitHub token
- downstream DB service role
- payment provider secret

このscaffoldでは実keyを置かず、`owner-age-recipient.example.txt` だけを置きます。public repo作成後にownerが実recipientへ差し替えます。
