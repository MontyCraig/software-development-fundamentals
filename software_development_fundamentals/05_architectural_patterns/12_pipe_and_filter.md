# Pipe and Filter Architecture

## Overview and Intent

The Pipe and Filter pattern structures a system as a series of processing
stages (filters) connected by channels (pipes). Each filter receives input,
performs a specific transformation, and passes the result to the next filter
through a pipe. Filters are independent, stateless, and unaware of their
position in the pipeline.

The intent is to achieve composability and reusability of processing steps.
Because each filter performs one well-defined transformation, filters can be
rearranged, added, removed, or replaced without affecting other filters in the
pipeline. This makes the system easy to understand, test, and extend.

Pipe and filter architectures are ubiquitous in data processing: command-line
tool pipelines, data ETL workflows, image processing chains, compiler phases,
and stream processing systems all follow this pattern. The simplicity of "data
in, transformed data out" makes it one of the most intuitive architectural
patterns.

## Structure and Components

### Component Diagram

```
  Input       Pipe       Filter      Pipe       Filter      Pipe       Output
  Source  ============> +--------+ =========> +--------+ =========>  Sink
    |                   |Validate|            |Transform            |
    |     raw data      | Data   | valid data | Data   | final data|
    +==================>+--------+==========>+--------+===========>+
                             |                    |
                        rejects invalid      applies rules

  Multi-branch Pipeline:

  Input ====> [Filter A] ====> [Filter B] ===+===> [Filter D] ====> Output 1
                                              |
                                              +===> [Filter E] ====> Output 2
```

### Key Components

1. **Filter**: A processing unit that reads data from its input pipe,
   performs a transformation, and writes results to its output pipe.
   Each filter is independent and stateless.

2. **Pipe**: A connector that transfers data between filters. May be
   synchronous (blocking) or asynchronous (buffered).

3. **Data Source**: The origin of data flowing into the pipeline (file,
   network stream, database query, user input).

4. **Data Sink**: The final destination of processed data (file, database,
   display, network endpoint).

5. **Pipeline**: The complete assembly of filters and pipes forming an
   end-to-end processing chain.

## How It Works

```
  Raw Data
     |
     v
  +--+--------+
  | Filter 1  |  1. Read raw input
  | (Parse)   |  2. Parse into structured format
  +--+--------+  3. Pass structured data downstream
     |
     v
  +--+--------+
  | Filter 2  |  4. Receive structured data
  | (Validate)|  5. Check business rules and constraints
  +--+--------+  6. Pass valid records, route invalid to error sink
     |
     v
  +--+--------+
  | Filter 3  |  7. Receive valid data
  | (Enrich)  |  8. Look up additional information
  +--+--------+  9. Attach enrichment to records
     |
     v
  +--+--------+
  | Filter 4  | 10. Receive enriched data
  | (Format)  | 11. Convert to output format
  +--+--------+ 12. Write to data sink
     |
     v
  Output Sink
```

## Pseudocode Example

```
// === FILTER INTERFACE ===

INTERFACE Filter:
    FUNCTION Process(input: DataStream) -> DataStream
END INTERFACE

// === PIPE (Connects Filters) ===

STRUCTURE Pipe:
    buffer: Queue of DataRecord
    capacity: Integer
END STRUCTURE

FUNCTION Pipe.Write(record: DataRecord):
    IF this.buffer.IsFull() THEN
        WaitUntilSpace(this.buffer)
    END IF
    this.buffer.Enqueue(record)
END FUNCTION

FUNCTION Pipe.Read() -> DataRecord:
    IF this.buffer.IsEmpty() THEN
        WaitUntilAvailable(this.buffer)
    END IF
    RETURN this.buffer.Dequeue()
END FUNCTION

// === PIPELINE BUILDER ===

STRUCTURE Pipeline:
    filters: List of Filter
END STRUCTURE

FUNCTION Pipeline.AddFilter(filter: Filter) -> Pipeline:
    Append(this.filters, filter)
    RETURN this
END FUNCTION

FUNCTION Pipeline.Execute(input: DataStream) -> DataStream:
    SET current = input
    FOR EACH filter IN this.filters DO
        SET current = filter.Process(current)
    END FOR
    RETURN current
END FUNCTION

// === CONCRETE FILTERS ===

// Filter 1: Parse raw CSV lines into structured records
STRUCTURE CsvParseFilter IMPLEMENTS Filter:
    delimiter: String
    headers: List of String
END STRUCTURE

FUNCTION CsvParseFilter.Process(input: DataStream) -> DataStream:
    SET output = NEW DataStream
    SET lineNumber = 0

    FOR EACH line IN input DO
        SET lineNumber = lineNumber + 1
        IF lineNumber = 1 THEN
            SET this.headers = Split(line, this.delimiter)
            CONTINUE
        END IF

        SET values = Split(line, this.delimiter)
        IF Length(values) != Length(this.headers) THEN
            output.WriteError({line: lineNumber, error: "Column count mismatch"})
            CONTINUE
        END IF

        SET record = NEW DataRecord
        FOR i = 0 TO Length(this.headers) - 1 DO
            SET record[this.headers[i]] = Trim(values[i])
        END FOR
        SET record["_sourceLineNumber"] = lineNumber
        output.Write(record)
    END FOR

    RETURN output
END FUNCTION

// Filter 2: Validate records against business rules
STRUCTURE ValidationFilter IMPLEMENTS Filter:
    rules: List of ValidationRule
END STRUCTURE

FUNCTION ValidationFilter.Process(input: DataStream) -> DataStream:
    SET output = NEW DataStream

    FOR EACH record IN input DO
        SET errors = EMPTY LIST

        FOR EACH rule IN this.rules DO
            SET result = rule.Validate(record)
            IF NOT result.isValid THEN
                Append(errors, result.message)
            END IF
        END FOR

        IF Length(errors) = 0 THEN
            SET record["_validationStatus"] = "VALID"
            output.Write(record)
        ELSE
            SET record["_validationStatus"] = "INVALID"
            SET record["_validationErrors"] = errors
            output.WriteError(record)
        END IF
    END FOR

    RETURN output
END FUNCTION

// Filter 3: Enrich records with additional data
STRUCTURE EnrichmentFilter IMPLEMENTS Filter:
    lookupService: LookupService
    enrichField: String
    lookupKey: String
END STRUCTURE

FUNCTION EnrichmentFilter.Process(input: DataStream) -> DataStream:
    SET output = NEW DataStream

    FOR EACH record IN input DO
        SET key = record[this.lookupKey]
        SET enrichment = this.lookupService.Lookup(key)

        IF enrichment IS NOT NULL THEN
            FOR EACH field IN enrichment.fields DO
                SET record[this.enrichField + "." + field.name] = field.value
            END FOR
        END IF

        output.Write(record)
    END FOR

    RETURN output
END FUNCTION

// Filter 4: Transform field values
STRUCTURE TransformFilter IMPLEMENTS Filter:
    transformations: Map of String to Function
END STRUCTURE

FUNCTION TransformFilter.Process(input: DataStream) -> DataStream:
    SET output = NEW DataStream

    FOR EACH record IN input DO
        FOR EACH fieldName, transformFn IN this.transformations DO
            IF record.HasField(fieldName) THEN
                SET record[fieldName] = transformFn(record[fieldName])
            END IF
        END FOR
        output.Write(record)
    END FOR

    RETURN output
END FUNCTION

// Filter 5: Aggregate/reduce records
STRUCTURE AggregationFilter IMPLEMENTS Filter:
    groupByField: String
    aggregations: List of Aggregation
END STRUCTURE

FUNCTION AggregationFilter.Process(input: DataStream) -> DataStream:
    SET groups = NEW Map
    SET output = NEW DataStream

    // Collect into groups
    FOR EACH record IN input DO
        SET key = record[this.groupByField]
        IF NOT groups.HasKey(key) THEN
            SET groups[key] = NEW List
        END IF
        Append(groups[key], record)
    END FOR

    // Apply aggregations to each group
    FOR EACH key, records IN groups DO
        SET result = NEW DataRecord
        SET result[this.groupByField] = key
        SET result["_count"] = Length(records)

        FOR EACH agg IN this.aggregations DO
            SET result[agg.outputField] = agg.Calculate(records)
        END FOR

        output.Write(result)
    END FOR

    RETURN output
END FUNCTION

// === BUILDING AND RUNNING A PIPELINE ===

FUNCTION BuildSalesPipeline() -> Pipeline:
    SET pipeline = NEW Pipeline

    // Step 1: Parse CSV
    pipeline.AddFilter(NEW CsvParseFilter(delimiter: ","))

    // Step 2: Validate
    SET validator = NEW ValidationFilter
    validator.AddRule(RequiredField("product_id"))
    validator.AddRule(RequiredField("quantity"))
    validator.AddRule(NumericRange("quantity", min: 1, max: 10000))
    validator.AddRule(RequiredField("sale_date"))
    validator.AddRule(DateFormat("sale_date", "YYYY-MM-DD"))
    pipeline.AddFilter(validator)

    // Step 3: Enrich with product details
    pipeline.AddFilter(NEW EnrichmentFilter(
        lookupService: ProductLookupService,
        enrichField: "product",
        lookupKey: "product_id"
    ))

    // Step 4: Transform
    pipeline.AddFilter(NEW TransformFilter(transformations: {
        "quantity": ParseInteger,
        "unit_price": ParseDecimal,
        "sale_date": ParseDate
    }))

    // Step 5: Add computed field
    pipeline.AddFilter(NEW ComputedFieldFilter(
        fieldName: "total",
        compute: FUNCTION(record):
            RETURN record["quantity"] * record["unit_price"]
        END FUNCTION
    ))

    RETURN pipeline
END FUNCTION

FUNCTION ProcessSalesReport():
    SET pipeline = BuildSalesPipeline()
    SET input = FileDataStream.Open("sales_data.csv")
    SET output = pipeline.Execute(input)

    SET writer = CsvWriter.Open("processed_sales.csv")
    FOR EACH record IN output DO
        writer.WriteRecord(record)
    END FOR
    writer.Close()

    LOG "Processed " + output.recordCount + " records"
    LOG "Errors: " + output.errorCount
END FUNCTION
```

## When to Use

- **Data processing pipelines** (ETL, log analysis, report generation)
  where data flows through sequential transformation stages.
- **Stream processing** where data arrives continuously and must be
  processed incrementally.
- **Compiler design** where source code passes through lexing, parsing,
  optimization, and code generation stages.
- **Image/signal processing** where transformations are applied in sequence
  (resize, crop, filter, compress).
- **Systems requiring composable processing** where different combinations
  of filters serve different use cases.

## When NOT to Use

- **Interactive applications** where the processing model is request-response
  rather than streaming data flow.
- **Systems with complex branching logic** where data flow depends heavily
  on content, making a linear pipeline awkward.
- **Stateful processing** that requires access to the full dataset before
  producing output (aggregation is possible but less natural).
- **Low-latency requirements** where the overhead of passing data between
  filters is unacceptable.

## Real-World Applications

| Domain | Example | Why This Pattern |
|--------|---------|-----------------|
| Data engineering | ETL pipeline | Sequential parse, validate, transform, load stages |
| DevOps | Log processing | Parse, filter, enrich, aggregate, alert |
| Media | Video transcoding | Decode, resize, encode, package |
| Compilers | Source code compilation | Lex, parse, optimize, generate |
| Network security | Packet inspection | Capture, decode, analyze, classify, alert |

## Trade-offs

### Advantages

- **Composability**: Filters can be mixed and matched freely.
- **Reusability**: Each filter can be used in multiple pipelines.
- **Testability**: Each filter is tested independently with sample data.
- **Parallelism**: Independent filters can run concurrently.
- **Simplicity**: Each filter has a single, clear responsibility.

### Disadvantages

- **Data copying**: Data is copied between filters (overhead).
- **Shared state difficulty**: Filters are stateless by design.
- **Error handling**: Errors must propagate through the pipeline.
- **Latency**: Data must pass through all filters sequentially.
- **Type safety**: Pipe data format must be agreed upon between filters.

## Comparison with Related Patterns

| Dimension | Pipe and Filter | Event-Driven | Layered |
|-----------|----------------|-------------|---------|
| Data flow | Linear/branching stream | Publish/subscribe | Top-down calls |
| Coupling | Very loose (data format only) | Loose (event schema) | Moderate (layer interfaces) |
| Processing model | Streaming | Reactive | Request-response |
| State | Stateless filters | Stateful consumers | Stateful layers |
| Best for | Data transformation | Event reactions | Business applications |

## Evolution and Variations

- **Linear Pipeline**: Simple chain of filters. Most common variant.
- **Branching Pipeline**: Filters can fork data to multiple downstream
  paths based on content or type.
- **Parallel Pipeline**: Multiple filter instances process data partitions
  concurrently for throughput.
- **Feedback Pipeline**: Output from later stages feeds back to earlier
  stages for iterative refinement.
- **Typed Pipes**: Pipes enforce type contracts between filters, catching
  incompatibilities at assembly time rather than runtime.

## Key Takeaways

1. Each filter should do one thing well. If a filter handles multiple
   concerns, split it into smaller filters.
2. Define a clear data format for pipes. All filters in a pipeline must
   agree on the shape of data flowing between them.
3. Error handling is a first-class concern. Decide early whether invalid
   records are dropped, routed to an error sink, or annotated and passed
   along.
4. Stateless filters enable parallelism. If a filter maintains state, it
   becomes a bottleneck and cannot be scaled horizontally.
5. Pipe and filter excels at data-centric workflows. For interactive or
   event-driven systems, other patterns are more appropriate.
