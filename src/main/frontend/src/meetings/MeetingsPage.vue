<template>
  <div>
    <new-meeting-form @added="addNewMeeting($event)"></new-meeting-form>

    <span v-if="meetings.length == 0">
               Brak zaplanowanych spotkań.
           </span>
    <h3 v-else>
      Zaplanowane zajęcia ({{ meetings.length }})
    </h3>

    <meetings-list :meetings="meetings"
                   :username="username"
                   @attend="addMeetingParticipant($event)"
                   @unattend="removeMeetingParticipant($event)"
                   @delete="deleteMeeting($event)"></meetings-list>
  </div>
</template>

<script>
    import NewMeetingForm from "./NewMeetingForm";
    import MeetingsList from "./MeetingsList";

    export default {
        components: {NewMeetingForm, MeetingsList},
        props: ['username'],
        data() {
            return {
                meetings: [],
				message: "",
				isError: false
            };
        },
        methods: {
            addNewMeeting(meeting) {
				this.clearMessage();
				this.$http.post('meetings', meeting)
					.then(response => {
						this.success('Spotkanie zostało utworzone');
					})
					.catch(response =>this.failure('Błąd przy tworzeniu spotkania. Kod odpowiedzi: ' + response.status));
				this.getMeetings();
			},
            addMeetingParticipant(meeting) {
                this.clearMessage();
				user = this.$http.get('participants/${username}');
				this.$http.post('meetings/${meeting.id}/participants',user)
					.then(response => {
						this.success(response.status)
					})
					.catch(response => this.failure('Błąd przy dodawaniu uczestnika. Kod odpowiedzi: ' + response.status));
				this.getMeetings();
            },
            removeMeetingParticipant(meeting) {
                meeting.participants.splice(meeting.participants.indexOf(this.username), 1);
            },
            deleteMeeting(meeting) {
                this.meetings.splice(this.meetings.indexOf(meeting), 1);
            },
			success(message) {
                this.message = message;
                this.isError = false;
            },
            failure(message) {
                this.message = message;
                this.isError = true;
            },
            clearMessage() {
                this.message = undefined;
            },
			getMeetings() {
				this.$http.get('meetings')
					.then (response => {
						this.meetings = response.body
					})
					.catch(response =>this.failure('Błąd przy tworzeniu spotkania. Kod odpowiedzi: ' + response.status));
			},
			getMeetingsParticipants() {
				for (meeting in this.meetings) {
					meeting.participants = [];
					this.$http.get('meetings/' + meeting.id + '/participants')
					.then (response => {
						meeting.participants = response.body
					})
					.catch(response =>this.failure('Błąd. Kod odpowiedzi: ' + response.status));
				}
			}
        },
		mounted() {
			this.getMeetings();
		}
    }
</script>
