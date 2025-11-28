# Mcp-server and GitHub Integration

This document outlines the various ways the `mcp-server` application is integrated with GitHub.

## 1. Fetching Documentation from GitHub

The `mcp-server` fetches documentation for resources directly from GitHub repositories. This is achieved by constructing a URL to the raw content of a markdown file in a specific GitHub repository.

- **URL Construction:** The application builds a URL using a base URL for raw GitHub content (`https://raw.githubusercontent.com/`) and appends the organization, repository, branch, and file path.
- **Fetching Content:** An HTTP GET request is sent to the constructed URL to retrieve the markdown content of the documentation.

This allows the application to display up-to-date documentation for various resources without having to store it locally.

## 2. Searching GitHub Repositories

`mcp-server` can search for repositories on GitHub using both the REST and GraphQL APIs.

- **REST API:** The application can search for repositories based on keywords, organizations, and other filters. This is used to find relevant repositories for a given query.
- **GraphQL API:** For more complex queries and to fetch more specific data about repositories, the application uses the GitHub GraphQL API.

## 3. Integration with GitHub Actions for CI/CD

There are indications of integration with GitHub Actions for Continuous Integration and Continuous Deployment (CI/CD). This is suggested by mentions of "Run GitHub Actions remotely on HCP Terraform" and "Use GitHub Actions with OIDC". This allows for automated workflows to build, test, and deploy the `mcp-server` application.

## 4. Mocking GitHub Releases for Testing

For testing purposes, the application includes functionality to create mock GitHub release data. This allows for testing of features that rely on GitHub release information without having to make actual calls to the GitHub API, making the tests more reliable and independent.

## 5. Fetching GitHub Repository Star Counts

The application has a feature to fetch and display the number of stars for a given GitHub repository. This is likely used to display repository popularity or to sort repositories by their star count.

## 6. GitHub Search in Detail

The `mcp-server` uses a sophisticated approach to search for information on GitHub, primarily for finding repositories. It uses the GitHub GraphQL API as its main search method and falls back to the REST API in certain situations.

### Primary Search Method: GraphQL API

The primary search functionality is handled by the `github_repo_search_graphql` function. This method is preferred due to the power and flexibility of the GraphQL API.

**Step-by-step process:**

1.  **Query Construction:** The function dynamically constructs a GraphQL query string. This query is tailored based on the search keywords, organizations, and other filters provided. The query is designed to fetch a rich set of information about the repositories, including their name, owner, description, star count, and more.

2.  **Authentication:** The search request is authenticated using a GitHub token. This is crucial for accessing the GraphQL API and for getting a higher rate limit. If no token is provided, the search might fail or be severely limited.

3.  **Making the Request:** The `github_graphql_request` helper function is used to send the GraphQL query to the GitHub API endpoint (`https://api.github.com/graphql`). This function handles the HTTP POST request, including setting the necessary headers (e.g., `Authorization` header with the token).

4.  **Response Parsing:** Upon receiving a response from the GitHub API, the application parses the JSON data to extract the list of repositories and other relevant information, like the total count of search results and pagination details.

5.  **Filtering and Processing:** The search results are further filtered and processed to match the specific requirements of the application. This might include filtering out archived or disabled repositories.

### Fallback Search Method: REST API

In cases where the GraphQL API cannot be used (e.g., when the GraphQL rate limit is hit and no token is available), the application falls back to the `github_repo_search_rest` function, which uses the standard GitHub REST API.

**Step-by-step process:**

1.  **Fallback Trigger:** This method is typically triggered as a fallback when the GraphQL search fails, especially due to rate limiting.

2.  **Query Construction:** A search query string is constructed using the provided keywords and filters. This query string is then used as a parameter for the REST API search endpoint.

3.  **Making the Request:** An HTTP GET request is sent to the GitHub REST API search endpoint (`https://api.github.com/search/repositories`).

4.  **Response Processing:** The JSON response from the REST API is parsed to extract the list of repositories. The data from the REST API might be less detailed than the GraphQL API, so additional processing might be needed to format it into a consistent structure.
