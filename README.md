# bitcoin-only-dev-bounties-spec
The site should give visitors an overview over all available Bitcoin bounties. We do not expect all bounty issuers to post their bounties themselves, so users can publish bounties linked to posts from Twitter or GitHub. If someone anonymous (Login with LNURL-Auth) wants to start a bounty the person should send the funds to OpenSats to ensure the payment is made if the task has been fulfilled. In cases of dispute separate moderators or OpenSats board decides if the bounty is paid out. 

Normal users also have the option to give OpenSats control over the funds to get higher trust. Users can also pledge a bounty to an already existing task/issue. They can add/clarify their own conditions which should not differ much from the existing conditions.

Furthermore users can post an issue/task they see the need for without adding a bounty. These act as a sort of fundraisers, where users who also think this issue is important can pledge funds. As soon as someone adds a bounty to an unfunded issue it is marked as a bounty and displayed more prominently. 


MVP:
- Login with Twitter, GitHub
- post issue
- add bounty to issue (1. from yourself, 2. from yourself + web announcement link, 3. from someone else + web announcement link (reference) )
- create bounty (= post issue + add bounty to this issue internally)
- main page (bounties): sort by total bounty amount, date
- second main page (unfunded issues): sorted by date (newest)
- an email address for disputes

Additional Features (most important to less important):

1)
- moderator functionality in UI
- upvotes 
- comments
- lnurl-auth login

2)
- give control over funds to OpenSats (payment functionality) → marked as “funding secured” 
- add dispute/claim 

### Rough enitity relationship diagram:

![er-diagram](https://user-images.githubusercontent.com/24638508/121535868-d3f92d00-ca02-11eb-9d9e-10c0af5dd9b6.png)

### SQL TABLE DESIGN
```
  await client.query(` CREATE TABLE IF NOT EXISTS People(
      id SERIAL PRIMARY KEY,
      name CHAR(64),
      url CHAR(128),
      UNIQUE(url)
  )`)

  await client.query(` CREATE TABLE IF NOT EXISTS Users(
    id SERIAL PRIMARY KEY,
    provider_id CHAR(128),
    provider_name CHAR(32),
    created_on TIMESTAMP NOT NULL,
    url CHAR(128) NOT NULL,
    privilege_level INT DEFAULT 0,
    UNIQUE(provider_id,provider_name)
  )
    `)

    await client.query(`CREATE TABLE IF NOT EXISTS Issues(
      id SERIAL PRIMARY KEY,
      user_id INTEGER NOT NULL REFERENCES Users(id),
      created_on TIMESTAMP NOT NULL,
      title CHAR(100) NOT NULL,
      link CHAR(128),
      description TEXT,
      PRIMARY KEY(id)
    )`)

  await client.query(`CREATE TABLE IF NOT EXISTS Bounties(
    issue_id INTEGER NOT NULL REFERENCES Issues(id),
    user_id INTEGER NOT NULL REFERENCES Users(id),
    amount INT NOT NULL;
    created_on TIMESTAMP NOT NULL,
    funding_secured BOOLEAN DEFAULT false,
    condition_text TEXT,
    PRIMARY KEY(issue_id,user_id)
  )
`)


  await client.query(`CREATE TABLE IF NOT EXISTS Comments(
    id SERIAL PRIMARY KEY,
    user_id INTEGER NOT NULL REFERENCES Users(id),
    text TEXT NOT NULL
    issue_id INTEGER REFERENCES Issues(id),
    comment_id INTEGER REFERENCES Comments(id)
  )
    `)

  await client.query(`CREATE TABLE IF NOT EXISTS Votes(
    user_id INTEGER NOT NULL REFERENCES Users(id),
    issue_id INTEGER NOT NULL REFERENCES Issues(id),
    PRIMARY KEY(user_id,issue_id),
    weight INTEGER DEFAULT 1
  )`)
´´´
