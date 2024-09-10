# Algo-Practice-Activity-Reduce-Map

### Part 1: Counting Words

#### Implementing `word_count_map`
The `word_count_map` function will map each document to a list of tuples where each tuple contains a word and the number `1`, representing one occurrence of that word. 

```python
def word_count_map(doc):
    # Split the document into words and return a list of (word, 1) tuples
    return [(word, 1) for word in doc.split()]
```

#### Implementing `word_count_reduce`
The `word_count_reduce` function will reduce the list of tuples for each word by summing up the occurrences.

```python
def word_count_reduce(word, counts):
    # Sum the counts for the given word
    return (word, sum(counts))
```

#### Testing `word_count_map` and `word_count_reduce`
Let's create some simple test functions to validate the above implementations.

```python
def test_word_count_map():
    doc = 'i am sam i am'
    expected_output = [('i', 1), ('am', 1), ('sam', 1), ('i', 1), ('am', 1)]
    assert word_count_map(doc) == expected_output, "word_count_map test failed!"
    print("word_count_map test passed!")

def test_word_count_reduce():
    word = 'am'
    counts = [1, 1]
    expected_output = ('am', 2)
    assert word_count_reduce(word, counts) == expected_output, "word_count_reduce test failed!"
    print("word_count_reduce test passed!")
```

**Now let's test the full map-reduce process with 'test_word_count'.**

```python
def run_map_reduce(map_f, reduce_f, docs):
    mapped = []
    for doc in docs:
        mapped.extend(map_f(doc))
    
    # Group by word
    from collections import defaultdict
    grouped = defaultdict(list)
    for word, count in mapped:
        grouped[word].append(count)
    
    # Reduce step
    reduced = []
    for word, counts in grouped.items():
        reduced.append(reduce_f(word, counts))
    
    return sorted(reduced)

def test_word_count():
    docs = ['i am sam i am', 'sam is ham']
    expected_output = [('am', 2), ('ham', 1), ('i', 2), ('is', 1), ('sam', 2)]
    assert run_map_reduce(word_count_map, word_count_reduce, docs) == expected_output, "test_word_count failed!"
    print("test_word_count passed!")
```

#### Work and Span of `word_count_reduce`
- **Work:** The work is \(O(n)\) where \(n\) is the number of occurrences of the word because we have to sum all the counts.
- **Span:** The span is \(O(\log n)\) assuming a parallel implementation because we can sum the counts in a parallel reduction.

#### Parallelization Challenge
The main issue with the sequential `counts` approach is that it involves a shared mutable state (`counts` dictionary). In a parallel context, multiple threads would have to access and update this shared state, leading to potential data races and requiring complex synchronization mechanisms. MapReduce, by contrast, avoids this issue by ensuring each map task works independently and by reducing after all mapping is complete.

### Part 2: Sentiment Analysis

#### Implementing `sentiment_map`
The `sentiment_map` function will map each document to a list of tuples where each tuple represents the occurrence of a positive or negative term.

```python
def sentiment_map(doc, positive_terms, negative_terms):
    words = doc.split()
    sentiment_tuples = []
    for word in words:
        if word in positive_terms:
            sentiment_tuples.append(('positive', 1))
        elif word in negative_terms:
            sentiment_tuples.append(('negative', 1))
    return sentiment_tuples
```

#### Testing `sentiment_map`

```python
def test_sentiment_map():
    doc = 'it was a terrible waste of time'
    positive_terms = []
    negative_terms = ['terrible', 'waste']
    expected_output = [('negative', 1), ('negative', 1)]
    assert sentiment_map(doc, positive_terms, negative_terms) == expected_output, "sentiment_map test failed!"
    print("sentiment_map test passed!")
```

#### Reusing `word_count_reduce`
I am reusing the `word_count_reduce` function from Part 1 for reducing the sentiment counts and also confirming that my results work by running test_sentiment. Here’s how we can test the complete sentiment analysis:

```python
def test_sentiment():
    docs = ['it was a terrible waste of time', 'this is an amazing experience']
    positive_terms = ['amazing']
    negative_terms = ['terrible', 'waste']
    expected_output = [('negative', 2), ('positive', 1)]
    
    # Run map-reduce for sentiment analysis
    result = run_map_reduce(lambda doc: sentiment_map(doc, positive_terms, negative_terms), word_count_reduce, docs)
    
    assert result == expected_output, "test_sentiment failed!"
    print("test_sentiment passed!")
```
