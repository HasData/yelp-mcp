---
description: What Yelp reviewers say about one business, grouped into themes
---

Summarise a business's reputation on Yelp.

Ask me for the business and the city if I have not given them.

Then:

1. Call `hasdata_yelp_search_getSearchResults` with `keyword` and `location` and take the `placeId` of the match. If several places share the name, show them and ask which one.
2. Call `hasdata_yelp_place_getPlaceDetails` for the rating, the review count, the price band, the categories and the hours.
3. Call `hasdata_yelp_reviews_getPlaceReviews` with `sortBy: "dateDesc"`, one page, and a second page only if the first is thin.
4. Group the complaints and the praise into themes, and put a count on each theme. Quote two or three short lines verbatim, with the star rating of the review they came from.
5. Strip the `[[HIGHLIGHT]]` markers before quoting anything.

Say how many reviews the summary rests on and over what period. Ten recent reviews and a thousand lifetime ratings tell different stories, and conflating them is the easiest way to mislead.
