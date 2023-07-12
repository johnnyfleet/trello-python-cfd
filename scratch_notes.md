# Scratch Notes

Following the outline on [this stackexchange site](https://webapps.stackexchange.com/questions/44278/is-it-possible-to-list-all-activity-for-a-given-date-range-in-trello) I was trying to simplify the amount of data that comes back when needing to backfill historical data.

## Simple Request

A straightforward get to check that the authentication environment variables are working

```bash
http GET "https://api.trello.com/1/boards/7Pr30Oly/actions" key==$TRELLO_KEY token==$TRELLO_TOKEN
```

## More Complex Request

This limits to some info based on a date (UTC) and then formats through JQ. Ideally would want to remove records based on some data.

```bash
http GET "https://api.trello.com/1/boards/7Pr30Oly/actions" key==$TRELLO_KEY token==$TRELLO_TOKEN "since==Jul 1 2023" "limit==1000" "filter==updateCard" |  jq 'group_by(.data.card.id) | map({key: (.[0].data.card | "\(.name) (\(.id))"), value: map({date, member: .memberCreator.fullName, list: .data.list.name, list_before: .data.listBefore.name, list_after: .data.listAfter.name, comment: .data.text}) }) | from_entries'
```

## Filter Data
This one works - filters out only the transitions between lists and then groups by cardid
```bash
http GET "https://api.trello.com/1/boards/7Pr30Oly/actions" key==$TRELLO_KEY token==$TRELLO_TOKEN "since==Jun 1 2023" "limit==1000" "filter==updateCard" |  jq '.[] | select(.data.listBefore != null) | [.] | group_by(.data.card.id) | map({key: (.[0].data.card | "\(.name) (\(.id))"), value: map({date, member: .memberCreator.fullName, list_before: .data.listBefore.name, list_after: .data.listAfter.name, comment: .data.text}) }) | from_entries'
```
