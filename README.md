# API Practice: TV Maze


###### You can find a page demonstrating this project here -> 

### My method of finishing this project
```js
"use strict";

const $showsList = $("#showsList");
const $episodesArea = $("#episodesArea");
const $episodesList = $("#episodesList");
const $searchForm = $("#searchForm");

/** Given a search term, search for tv shows that match that query.
 *
 *  Returns (promise) array of show objects: [show, show, ...].
 *    Each show object should contain exactly: {id, name, summary, image}
 *    (if no image URL given by API, put in a default image URL)
 */

async function getShowsByTerm(term) {
    // ADD: Remove placeholder & make request to TVMaze search shows API.
    const request = await axios.get('https://api.tvmaze.com/search/shows/', { params: { q: term } });
    const shows = request.data;
    // console.log(request);
    // console.log(shows);
    const finalreturn = shows.map(function(value){
        let img;
        try{
            img = value.show.image.medium;
        }
        catch(e){
            img = 'https://tinyurl.com/tv-missing';
        }
        return {
            id: value.show.id,
            name: value.show.name,
            summary: value.show.summary,
            image: img
        }
    });
    return finalreturn;
}


/** Given list of shows, create markup for each and to DOM */

function populateShows(shows) {
    $showsList.empty();

    for (let show of shows) {
        const $show = $(
            `<div data-show-id="${show.id}" class="Show col-md-12 col-lg-6 mb-4">
         <div class="media">
           <img
              src=${show.image}
              alt=${show.name}
              class="w-25 me-3">
           <div class="media-body">
             <h5 class="text-primary">${show.name}</h5>
             <div><small>${show.summary}</small></div>
             <button class="btn btn-outline-light btn-sm Show-getEpisodes" data-show-id=${show.id}>
               Episodes
             </button>
           </div>
         </div>
       </div>
      `);

        $showsList.append($show);
    }
}


/** Handle search form submission: get shows from API and display.
***     Hide episodes area (that only gets shown if they ask for episodes)
***/

async function searchForShowAndDisplay() {
    const term = $("#searchForm-term").val();
    const shows = await getShowsByTerm(term);

    $episodesArea.hide();
    populateShows(shows);
}

$searchForm.on("submit", async function (evt) {
    evt.preventDefault();
    await searchForShowAndDisplay();
});

$showsList.on('click', async function(evt){
    // evt.preventDefault();
    if (evt.target.localName === 'button'){
        const id = (evt.target.attributes[1].value);
        const episodes = (await getEpisodesOfShow(id));
        populateEpisodes(episodes);
    }
});

/** Given a show ID, get from API and return (promise) array of episodes:
 *      { id, name, season, number }
 */

async function getEpisodesOfShow(id) { 
    const request = await axios.get(`https://api.tvmaze.com/shows/${id}/episodes`);
    const episodes = request.data;
    // console.log(episodes);
    return episodes.map(function(episode){
        return {
            id: episode.id,
            name: episode.name,
            season: episode.season,
            number: episode.number
        }
    });
}

/** Write a clear docstring for this function 
 * populateEpisodes() clears the episode list if it exists
 * checks if the actual list is zero in which case will just throw an alert to the user that there wasn't any episode information found
 * if there is at least 1 episode it will create an 'li' and a 'b' (bold) element which bolds the episodes name and appends it to the li
 * li then appends the episodes information (episode.season) and (episode.number) (episode)
 * appends li to $episodesList 'ul'
 * lastly $episodesArea is shown
*/


function populateEpisodes(episodes) {
    $episodesList.empty();
    if (episodes.length === 0){ // checks if the actual list is zero in which case will just throw an alert to the user that there wasn't any episode information found
        alert('There\'s no episode information for this show :(');
    }
    else{
        for (let episode of episodes){
            // console.log(`${episode.name} (season ${episode.season}, episode ${episode.number})`); // for debugging
            const episodeLi = document.createElement('li');
            const bold = document.createElement('b');
            
            // bold list element
            bold.innerText = `${episode.name} `;
            episodeLi.append(bold)

            // episode list element
            episodeLi.innerText += `(season ${episode.season}, episode ${episode.number})`;
            $episodesList.append(episodeLi);

            $episodesArea.show(); // shows the episode area list
        }
    }
}

```