---
layout: post
title:  "VSCode Devcontainers for an MMO Emulator"
date:   2024-07-02 09:33:13 -0400
---

TF-IDF (Term Frequency-Inverse Document Frequency) and Cosine Similarity are foundational techniques in natural language processing used to quantify and compare text data. TD-IDF identifies significant words in a set of documents by balancing how often a word appears in a single document against how common it is across the entire dataset. Cosine Similarity then measures how similar two pieces of text are by calculating the angle between their vector representations. While these methods are simpler than the complex embeddings employed by modern LLMs, they remain effective for straightforward text analysis tasks.

In this project, MARC records from the Library of Congress are processed and cleaned into a structured JSON format using PyMARC. Upon server initialization, a TD-IDF matrix is generated to establish vector representations of the records. When a user inputs a book title, Cosine Similarity calculates the closeness of the input to existing records, enabling quick, computationally efficient book recommendations without the overhead of more complex neural network models.

## Environment
- Windows
- Docker Desktop
- WSL2
- VSCode
- PyMARC
- SciKit-Learn

The production application is hosted on a DigitalOcean droplet, running containers via Docker Compose. A Python Flask app serves as the frontend, reverse proxied by Nginx with Let’s Encrypt SSL for secure connections. LOC MARC data was downloaded in 43 separate .mrc files, and PyMARC was used to extract specific tags into structured JSON data. A TF-IDF matrix is computed on all the book data, allowing Cosine Similarity to identify text similar to user search queries.

Here we can see some of the code use to extract the MARC data from the .mrc files; more about MARC 21 format here [MARC 21 Format](https://www.loc.gov/marc/bibliographic/){:target="_blank"}

```
# Open each file and process it
with open(file_path, 'rb') as fh:
    reader = MARCReader(fh)
    for record in reader:
        book = {
            'isbn': '',
            'author': '',
            'title': ''
        }

        # Requiring English language (some foreign books have English 520s)
        lang = record['008']
        if lang and lang.data[35:38] != 'eng':
            continue

        # Extract ISBN
        book['isbn'] = get_isbn_from_record(record)

        # Extract author and title
        book['author'] = record.get_fields('100')[0].get('a', '') \
            if record.get_fields('100') else ''
        book['title'] = record.get_fields('245')[0].get('a', '') \
            if record.get_fields('245') else ''

        # Extract description from 520
        description = record.get_fields('520')[0].get('a', '') \
            if record.get_fields('520') else ''

        # Requiring 520 (better quality data)
        if description == '':
            continue

        description = description + ' ' + record.get_fields('520')[0].get('b', '') \
            if record.get_fields('520') else ''

        # Extract 650 subjects
        subjects = record.get_fields('650')
        for subject in subjects:
            description = description + ' ' + subject.get('a', '')

        book['description'] = description

        # Check for ISBN and description before adding
        if book['isbn'] and book['description']:
            book['title'] = clean_book_title(book['title'])
            books.append(book)

```

The most important fields from our MARC data extraction are the title, author, subjects, and annotations. These fields are what’s computed into the TF-IDF matrix and then ultimately compared against for similarity searches.

The public available MARC data used for this contains over 2 million books, however after some filtering for English only and books that have these fields we're left over with about 400,000 in our collection.

At this point the scikit-learn python library does the remaining work, by computing the TD-IDF matrix for every book and providing the cosine simularity functions.

```
def compute_tfidf():
    global tfidf_matrix
    logging.info(f'Compute the TF-IDF matrix')
    tfidf = TfidfVectorizer(stop_words='english')
    book_descriptions = [book['description'] for book in books]
    tfidf_matrix = tfidf.fit_transform(book_descriptions)
```

```
# Takes a title and computes cosine similarity to return top 10 results
def get_book_recommendation(title):
    logging.info(f'Get book recommendation for title: {title}')
    book_data = next((book for book in books if book['title'] == title), None)
    # Get the feature vector for the target book
    target_vec = tfidf_matrix[books.index(book_data)]
    # Get the pairwise similarity scores of all books with the target book
    sim_scores = list(enumerate(cosine_similarity(target_vec, tfidf_matrix)[0]))
    # Sort the books based on the similarity scores
    sim_scores = sorted(sim_scores, key=lambda x: x[1], reverse=True)
    # Get the scores of the 10 most similar books
    sim_scores = sim_scores[1:11]
    # Get the book indices
    book_indices = [i[0] for i in sim_scores]
    # Return the top 10 most similar books
    return [books[idx] for idx in book_indices]
```

![Example book search](/assets/20240702-bbb.png)