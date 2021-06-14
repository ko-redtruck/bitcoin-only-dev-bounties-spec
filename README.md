# bitcoin-only-dev-bounties-spec
### WIP Implementation: https://github.com/ko-redtruck/bitcoin-only-open-source-bounties-express
The site should give visitors an overview over all available Bitcoin bounties. We do not expect all bounty issuers to post their bounties themselves, so users can publish bounties linked to posts from Twitter or GitHub. If someone anonymous (Login with LNURL-Auth) wants to start a bounty the person should send the funds to OpenSats to ensure the payment is made if the task has been fulfilled. In cases of dispute separate moderators or OpenSats board decides if the bounty is paid out. 

Normal users also have the option to give OpenSats control over the funds to get higher trust. Users can also pledge a bounty to an already existing task/issue. They can add/clarify their own conditions which should not differ much from the existing conditions.

Furthermore users can post an issue/task they see the need for without adding a bounty. These act as a sort of fundraisers, where users who also think this issue is important can pledge funds. As soon as someone adds a bounty to an unfunded issue it is marked as a bounty and displayed more prominently. 


MVP:
- Login with Twitter (:x:), GitHub (:heavy_check_mark:)
- post issue (:heavy_check_mark:)
- add bounty to issue (1. from yourself, 2. from yourself + web announcement link, 3. from someone else + web announcement link (reference) ) (:x:)
- create bounty (= post issue + add bounty to this issue internally) (:x:)
- main page (bounties): sort by total bounty amount, date (:x:)
- second main page (unfunded issues): sorted by date (newest) (:x:)
- an email address for disputes (:x:)

Additional Features (most important to less important):

1)
- moderator functionality in UI (:x:)
- upvotes (You can only upvote comments, if you want to "upvote" an issue/bounty add your own bounty to signal support) (:x:)
- comments (:x:)
- lnurl-auth login (:x:)

2)
- give control over funds to OpenSats (payment functionality) → marked as “funding secured” (:x:)
- add dispute/claim (:x:)

### Rough/incomplete enitity relationship diagram:

![er-diagram](https://user-images.githubusercontent.com/24638508/121535868-d3f92d00-ca02-11eb-9d9e-10c0af5dd9b6.png)

### SQL TABLE DESIGN
```
await client.query(` CREATE TABLE IF NOT EXISTS Identities(
      url VARCHAR(128),
      name VARCHAR(64),
      PRIMARY KEY(url),
      UNIQUE(name)
  )`)

  await client.query(` CREATE TABLE IF NOT EXISTS Users(
    id SERIAL PRIMARY KEY,
    provider_id VARCHAR(128),
    provider_name VARCHAR(32),
    created_on TIMESTAMP NOT NULL,
    identity_url VARCHAR(128) REFERENCES Identities(url),
    privilege_level INT DEFAULT 0,
    UNIQUE(provider_id,provider_name)
  )
    `)

  await client.query(`CREATE TABLE IF NOT EXISTS Issues(
    id SERIAL PRIMARY KEY,
    user_id INTEGER NOT NULL REFERENCES Users(id),
    created_on TIMESTAMP NOT NULL,
    title VARCHAR(100) NOT NULL,
    link VARCHAR(128),
    description TEXT,
    UNIQUE(title)
  )`)

  await client.query(`CREATE TABLE IF NOT EXISTS Bounties(
    issue_id INTEGER NOT NULL REFERENCES Issues(id),
    user_id INTEGER NOT NULL REFERENCES Users(id),
    identity_url VARCHAR(128) NOT NULL REFERENCES Identities(url),
    amount INT NOT NULL,
    created_on TIMESTAMP NOT NULL,
    funding_secured BOOLEAN DEFAULT false,
    announchment_link VARCHAR(128),
    condition_text TEXT,
    payed_out_to VARCHAR(128) REFERENCES Identities(url),
    PRIMARY KEY(issue_id,user_id,identity_url)
  )
`)


  await client.query(`CREATE TABLE IF NOT EXISTS Comments(
    id SERIAL PRIMARY KEY,
    user_id INTEGER NOT NULL REFERENCES Users(id),
    text TEXT NOT NULL,
    issue_id INTEGER REFERENCES Issues(id),
    comment_id INTEGER REFERENCES Comments(id)
  )
    `)

  await client.query(`CREATE TABLE IF NOT EXISTS Votes(
    user_id INTEGER NOT NULL REFERENCES Users(id),
    comment_id INTEGER NOT NULL REFERENCES Comments(id),
    PRIMARY KEY(user_id,comment_id),
    weight INTEGER DEFAULT 1
  )`)
```

### Urls
- /sign-up
- /auth/login 
- /auth/login/Github
- /auth/login/Twitter
- /auth/login/LNURL-Auth

- /post/issue (without bounty) [GET,POST]
- /post/issue/close [POST]
- /post/bounty (issue with bounty) [GET,POST]
- /post/bounty/pay [POST] --> marked as payed out
- /post/bounty/add [POST]
- /post/comment [POST]
- /post/upvote [POST]

- /issues [GET]
- /issues&bounty=true [GET]
- /issues/<id> [GET]
