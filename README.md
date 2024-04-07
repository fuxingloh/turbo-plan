# Turbo Plan

Plan turbo run tasks in a separate action to run them parallel in a matrix job.

## How to use?

Separate your tests into 3 steps:

1. `Test [plan]` using `fuxingloh/turbo-plan` to plan the tasks for the matrix job. 
2. `Test [run]` run the tasks in parallel using the matrix job.
3. `Test [completed]` wait for the matrix job to complete and check if any of the tasks failed.

```yml
name: CI

on:
  pull_request:
    branches: [ main ]

# You only need read permissions to use this action.
permissions:
  contents: read
  packages: read

jobs:
  test_plan:
    name: Test [plan]
    runs-on: ubuntu-latest
    outputs:
      # Output the packages to be used in the matrix job
      packages: ${{ steps.plan.outputs.packages }}
    steps:
      - # You must check out the repository!
      - uses: actions/checkout@v4

      - uses: fuxingloh/turbo-plan@v1
        id: plan
        with:
          # Provide the command you want to run
          task: test

  test_run:
    name: Test [run]
    needs: test_plan
    runs-on: ubuntu-latest
    strategy:
      fail-fast: false
      matrix:
        # Pass the packages to the matrix job
        package: ${{ fromJSON(needs.test_plan.outputs.packages) }}
    steps:
      - uses: actions/checkout@v4

      # Make sure to run the command with the same arguments as the plan step
      - run: npx turbo run test --filter=${{ matrix.package }}

  test_completed:
    name: Test [completed]
    runs-on: ubuntu-latest
    needs: test_run
    if: always()
    steps:
      - run: |
          if ${{ contains(needs.*.result, 'failure') || contains(needs.*.result, 'skipped') || contains(needs.*.result, 'cancelled') }} ; then          
            exit 1
          fi
```