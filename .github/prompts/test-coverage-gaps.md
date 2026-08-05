Identify the most valuable missing tests in this repository.

Backend tests live under `apps/api/tests` and run with pytest inside the stack
defined by `docker-compose-test.yml`. See `apps/api/tests/TESTING_GUIDE.md` for
conventions and fixtures.

Rank by risk rather than by raw coverage percentage. Prioritize:

1. Code paths handling authentication, permissions, or access control.
2. Data mutation paths where a bug corrupts or destroys user data.
3. Complex conditional logic with many branches and no corresponding test.
4. Recently changed code with no accompanying test.

For each gap, name the file and function, explain what breaks if it regresses, and
sketch the specific test case to add including the inputs that matter. Propose at
most ten, ordered by risk. Do not write the tests.
