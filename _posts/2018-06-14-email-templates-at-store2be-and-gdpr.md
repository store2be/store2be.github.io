---
date: 2018-06-14
categories:
- Email
- Sendwithus
- MJML
- Typescript
- React
title: Email templates at store2be and GDPR — How we migrated away from Sendwithus
author_staff_member: 01_peter
medium_link: https://medium.com/store2be-tech/email-templates-at-store2be-and-gdpr-how-we-migrated-away-from-sendwithus-e4350b2c24ca
practical_dev_link: https://dev.to/peterfication/email-templates-at-store2be-and-gdprhow-we-migrated-away-from-sendwithus-4go7
---

When [store2be](https://www.store2be.com) started 3 years ago we were searching for a nice way to handle email templating and sending. I came across [Sendwithus](https://www.sendwithus.com), an email template service that connects to a lot of different email providers, like SendGrid, Mailjet, etc.

We decided to use Sendwithus as it decoupled the email templating from our main application and allowed non developers to handle email template changes. Furthermore, it was very helpful to have different email sending providers integrated automatically. Once, we had to switch the email provider and it took a matter of minutes with Sendwithus.

Now that the GDPR is coming into effect we have to evaluate all the services we use and check whether they are compliant. In February, Sendwithus informed its users about their way of handling GDPR compliance:

> Unfortunately, we will not be making Sendwithus GDPR compliant.

This was a bummer for us. Although they now provide a new, compliant service, back then we started looking for a solution right away hearing from Sendwithus that they would not be attempting GDPR compliance. So we tried to find another service that matched our requirements but we weren’t successful.

---

At store2be, we are very keen on code quality and the tools around it (testing, linting, etc.). This was always a problem with Sendwithus. It kind of worked, but we were never sure whether we were going to break something and reviews only happened visually and not by looking at the actual code. Also, there was no nice Git history of the changes. Finally, there were a lot of hacks to get around the limitations of the templating possibilities of Sendwithus, eg. regarding snippets.

In the end we decided to move email templating into the hands of the developers again. The main reason for that might be the fact that Mailjet open sourced its email template markup language, [MJML](https://mjml.io), which makes writing HTML email templates super easy. In the frontend we are mainly developing with React in Typescript and Jest for tests. This seemed like a perfect fit for this project regarding code quality, testability and ease of use.

Of course we lose one very important attribute with that approach: All email template changes have to be done by developers again.

---

The open source project [Maily](https://github.com/inventid/maily) provided a lot of inspiration on how to get started with this service ([here is a Medium post about it from the creator of Maily](https://medium.com/@Rogier.Slag/creating-emails-with-the-maily-api-a-how-to-part-1-7f63306a7ad4)). Unfortunately, it is not maintained anymore and my issues and PRs are not being tackled. But in its core, Maily is just one file that creates the express server. So we copied this file into our repository and adjusted it to our needs (moving it to Typescript, satisfying the linter, updating MJML, adding more functionality).

This is what we are working with now:

- **Typescript:** All our code for the email templates is in Typescript. Hence, a lot of bugs are caught early.
- **Linter:** We use TSLint to comply to a coding standard we like.
- **Prettier:** We use Prettier to format our code. No discussions about each individual coding style.
- **Testing:** All components (snippets and email templates) are unit and snapshot tested. This means every developer feels confident about changing an email template. Furthermore, we use [lorikeet](https://github.com/cetra3/lorikeet) for integration tests. This adds an additional layer of safety we hadn’t thought about in the beginning.
- **Localization:** We use a very simple approach where each email template has a JSON file with the keys for each language we want to support. So the actual React component does not contain any literals but uses the translate function that reads this JSON file. Both, TXT and HTML templates use the same translations which reduces the possibility of inconsistencies.
- **Preview:** For development, you make GET request to the local express server (without hot reloading at the moment) to see a preview of the email. Online, the product team can do the same with the staging or production server. Furthermore, we have Swagger definitions for the email templates that can be transformed into Postman collections which makes the life of the product team even easier.
- **Review:** All code at store2be is reviewed. This also applies to the new email template service.


All in all, we are very happy with our decision of developing the email template service ourselves. Email templates are finally fun to work with.

Here is how an email template could look like now:

```typescript
import { generateFetchLocale } from 'lib/utils'
import * as React from 'react'

import Button from 'templates/html/snippets/Button'
import Closing from 'templates/html/snippets/Closing'
import Footer from 'templates/html/snippets/Footer'
import FullWidthBorder from 'templates/html/snippets/FullWidthBorder'
import Greeting from 'templates/html/snippets/Greeting'
import Header from 'templates/html/snippets/Header'
import Layout from 'templates/html/snippets/Layout'
import Text from 'templates/html/snippets/Text'
import Title from 'templates/html/snippets/Title'
import locales = require('templates/locales/Welcome.json')

const Welcome: React.SFC<WelcomeProps> = props => {
  const link = props.link || 'https://www.store2be.com'
  const user = props.user || { title: '', lastname: '' }
  const { locale } = props
  const fetchLocale = generateFetchLocale(locale, locales)
  return (
    <Layout env={props.env}>
      <Header />
      <Title>{fetchLocale('title')}</Title>

      <Greeting locale={locale} lastname={user.lastname} title={user.title} />

      <Text>{fetchLocale('welcome_please_confirm')}</Text>

      <Button link={link}>{fetchLocale('button')}</Button>

      <Text>
        {fetchLocale('button_not_working') + ' '}
        <a href={link}>{link}</a>
      </Text>

      <Closing locale={locale} />

      <FullWidthBorder />

      <Footer locale={locale} />
    </Layout>
  )
}

export default Welcome
```

---

![Example email preview.]({{ site.url }}/images/email-template.jpg)
